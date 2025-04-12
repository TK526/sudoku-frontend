<script setup lang="ts">
import { ref, reactive, computed, onMounted, onUnmounted, watch } from 'vue';

// Import Components
import SudokuGrid from './components/SudokuGrid.vue';
import Controls from './components/Controls.vue';
import NumberPad from './components/NumberPad.vue';
import Leaderboard from './components/Leaderboard.vue';
import GameEndModal from './components/GameEndModal.vue';

// Import Services and Types
import api from './services/api';
import type {
    CellPosition,
    LeaderboardData
} from './types';

// --- State ---
const gameId = ref<string | null>(null);
const difficulty = ref<string>('beginner');
const grid = ref<number[][]>([]);
const initialGrid = ref<number[][]>([]);
const selectedCell = ref<CellPosition | null>(null);
const errors = ref<number>(0);
const hintsUsed = ref<number>(0);
const maxHints = ref<number>(10); // Constant, could be fetched if dynamic
const visibleCount = ref<number>(0);
const gameStartTime = ref<number | null>(null); // Store as timestamp (milliseconds)
const elapsedTime = ref<number>(0); // Store as seconds
const timerInterval = ref<number | null>(null); // Stores interval ID (number in Node/browser)
const isPaused = ref<boolean>(false);
const isCompleted = ref<boolean>(false);
const isLoading = ref<boolean>(false); // Global loading state for major actions
const showGameEndModal = ref<boolean>(false);
const finalScore = ref<number | null>(null);
const errorCells = reactive<Set<string>>(new Set()); // 'row-col' format
const hintCells = reactive<Set<string>>(new Set()); // 'row-col' format

const leaderboardData = ref<LeaderboardData | null>(null);

// --- Computed Properties ---
const gameInProgress = computed<boolean>(() => gameId.value !== null && !isCompleted.value);
const hintsRemaining = computed<number>(() => maxHints.value - hintsUsed.value);
const gridDimension = computed<number>(() => grid.value?.length || 9);

const digitCounts = computed<{ [key: number]: number }>(() => {
    const counts: { [key: number]: number } = {};
    for (let i = 1; i <= 9; i++) {
        counts[i] = 0;
    }
    if (!grid.value || grid.value.length === 0) return counts;
    for (let r = 0; r < gridDimension.value; r++) {
        for (let c = 0; c < gridDimension.value; c++) {
            const val = grid.value[r]?.[c];
            if (val >= 1 && val <= 9) {
                counts[val]++;
            }
        }
    }
    return counts;
});

// --- Methods ---

// Fetch Leaderboard Data
const fetchLeaderboard = async (): Promise<void> => {
    // Avoid setting global isLoading maybe, handle in Leaderboard component?
    // Let's keep it simple for now.
    isLoading.value = true;
    try {
        const response = await api.getLeaderboard();
        leaderboardData.value = response.data;
    } catch (err) {
        console.error("Failed to fetch leaderboard:", err);
        // Handle error - maybe show a message to the user
    } finally {
         isLoading.value = false;
    }
};

// Start a New Game
const startGame = async (selectedDifficulty: string): Promise<void> => {
    if (isLoading.value) return;
    console.log("[Debug] startGame - Attempting to start game...");
    isLoading.value = true;
    resetGameState(); // Reset state before starting
    difficulty.value = selectedDifficulty;

    try {
        console.log("[Debug] startGame - Calling /game/create API...");
        const response = await api.createGame({ difficulty: difficulty.value, gridDimension: 9 }); // Assuming 9x9 always
        const { gameId: newGameId, hiddenGrid, visibleCount: initialVisibleCount } = response.data;
        console.log(`[Debug] startGame - Received gameId from backend: ${newGameId}`);

        if (!newGameId) throw new Error("Backend did not return a game ID.");

        gameId.value = newGameId;
        // Deep copy the grids
        grid.value = hiddenGrid.map(row => [...row]);
        initialGrid.value = hiddenGrid.map(row => [...row]);
        visibleCount.value = initialVisibleCount;
        gameStartTime.value = Date.now(); // Record start time

        console.log(`[Debug] startGame - Game state updated. gameId.value is now: ${gameId.value}`);
        startTimer(); // Start the game timer

    } catch (err: any) {
        console.error("[Debug] startGame - Error during game creation:", err);
        alert(`Error starting game: ${err.response?.data?.message || err.message || 'Unknown error'}`);
        resetGameState(); // Ensure state is reset on error
        console.log("[Debug] startGame - Game state reset due to error.");
    } finally {
        console.log("[Debug] startGame - Entering finally block.");
        isLoading.value = false;
        console.log("[Debug] startGame - isLoading set to false.");
    }
};

// Reset Game State
const resetGameState = (): void => {
    console.log("[Debug] resetGameState - Resetting game state...");
    stopTimer();
    gameId.value = null;
    grid.value = [];
    initialGrid.value = [];
    selectedCell.value = null;
    errors.value = 0;
    hintsUsed.value = 0;
    visibleCount.value = 0;
    gameStartTime.value = null;
    elapsedTime.value = 0;
    isPaused.value = false;
    isCompleted.value = false;
    showGameEndModal.value = false;
    finalScore.value = null;
    errorCells.clear();
    hintCells.clear();
    // Keep selected difficulty for potentially starting again easily
};

// Handle Cell Selection
const handleCellSelect = (position: CellPosition): void => {
    if (isPaused.value || isCompleted.value) return; // Don't allow selection if paused/completed

    // Check if the cell is pre-filled using initialGrid
    if (initialGrid.value[position.row]?.[position.col] !== 0) {
        selectedCell.value = null; // Deselect if pre-filled
        return;
    }
    selectedCell.value = position;
    errorCells.delete(`${position.row}-${position.col}`); // Clear error on select
};

// Handle Number Input (from Keyboard)
const handleNumberInput = async (value: number): Promise<void> => {
    if (!selectedCell.value || isLoading.value || isCompleted.value || isPaused.value) return;

    const { row, col } = selectedCell.value;
    if (initialGrid.value[row]?.[col] !== 0) return; // Double check pre-filled

    // Assume hints are permanent for now, prevent overwrite
    if (hintCells.has(`${row}-${col}`)) {
         console.log("Cannot overwrite hint cell.");
         return;
    }

    const oldValue = grid.value[row][col];
    if (oldValue === value) return; // No change needed

    grid.value[row][col] = value; // Optimistic UI update

    // isLoading is handled by checkGameCompletion if needed
    // isLoading.value = true; // Avoid setting here

    try {
        console.log(`[Debug] handleNumberInput - Calling api.checkValue for (${row},${col}) = ${value}`);
        const response = await api.checkValue({
        gameId: gameId.value!, // Use non-null assertion if sure gameId exists here
        row: row,
        column: col, // <--- RENAMED this property
        value: value
    });
    
        console.log("[Debug] handleNumberInput - api.checkValue response received:", response.data);

        const { correct, visibleValuesCount: newVisibleCount, errors: newErrors } = response.data;

        visibleCount.value = newVisibleCount;
        errors.value = newErrors; // Update errors based on backend response

        const cellKey = `${row}-${col}`;
        if (correct) {
            errorCells.delete(cellKey);
        } else {
            errorCells.add(cellKey);
        }

        // Check for potential completion
        if (correct && newVisibleCount >= 81) {
            console.log("[Debug] handleNumberInput - Condition met, calling checkGameCompletion...");
            await checkGameCompletion();
            console.log("[Debug] handleNumberInput - Returned from checkGameCompletion call.");
        }
    } catch (err: any) {
        console.error("[Debug] handleNumberInput - Error during checkValue API:", err);
        alert(`Error checking value: ${err.response?.data?.message || err.message || 'Unknown error'}`);
        grid.value[row][col] = oldValue; // Revert optimistic update
        // If it was previously correct, do we re-add error? No, keep simple.
    } finally {
        // Ensure isLoading is false if it was ever set to true during this flow
        // isLoading.value = false; // Not setting isLoading in this func currently
    }
};

// Clear Selected Cell (from Keyboard)
const clearSelectedCell = (): void => {
    if (!selectedCell.value || isLoading.value || isCompleted.value || isPaused.value) return;

    const { row, col } = selectedCell.value;
    const cellKey = `${row}-${col}`;

    // Only allow clearing user-entered cells (not pre-filled, not hints)
    if (initialGrid.value[row]?.[col] === 0 && !hintCells.has(cellKey) && grid.value[row]?.[col] !== 0) {
        grid.value[row][col] = 0; // Visual clear only
        errorCells.delete(cellKey); // Clear error state
        // Note: visibleCount & backend errors are not updated here
    } else {
        console.log(`Cannot clear cell ${row}-${col}: Pre-filled, Hinted, or already empty.`);
    }
};

// Request a Hint
const requestHint = async (): Promise<void> => {
    if (isLoading.value || isCompleted.value || hintsRemaining.value <= 0 || isPaused.value || !gameId.value) return;

    isLoading.value = true;
    console.log("[Debug] requestHint - Requesting hint...");

    try {
        const response = await api.useHint({ gameId: gameId.value });
        console.log("[Debug] requestHint - Hint response received:", response.data);
        const hintData = response.data;

        if (hintData.error) {
            alert(`Hint Error: ${hintData.error}`);
            if (hintData.hintsUsed !== undefined) hintsUsed.value = hintData.hintsUsed;
            if (hintData.visibleValuesCount !== undefined) visibleCount.value = hintData.visibleValuesCount;
        } else if (hintData.rowIndex !== undefined && hintData.columnIndex !== undefined && hintData.value !== undefined) {
            // Success case
            grid.value[hintData.rowIndex][hintData.columnIndex] = hintData.value;
            hintsUsed.value++; // Increment frontend count
            if (hintData.visibleValuesCount !== undefined) visibleCount.value = hintData.visibleValuesCount;

            const cellKey = `${hintData.rowIndex}-${hintData.columnIndex}`;
            hintCells.add(cellKey); // Mark as hint
            errorCells.delete(cellKey); // Remove error if hint overwrites it

            // Check for potential completion after hint
            if (visibleCount.value >= 81) {
                console.log("[Debug] requestHint - Condition met after hint, calling checkGameCompletion...");
                await checkGameCompletion();
            }
        }
    } catch (err: any) {
        console.error("[Debug] requestHint - Failed to get hint:", err);
        alert(`Error getting hint: ${err.response?.data?.message || err.message || 'Unknown error'}`);
    } finally {
        isLoading.value = false;
        console.log("[Debug] requestHint - Finished hint request.");
    }
};

// Check Game Completion Status with Backend
const checkGameCompletion = async (): Promise<void> => {
    console.log("[Debug] Attempting to enter checkGameCompletion function.");
    if (!gameId.value) {
        console.log("[Debug] checkGameCompletion - Exiting early due to missing gameId.");
        return;
    }
    console.log("[Debug] checkGameCompletion - Passed gameId check.");

    isLoading.value = true; // Indicate loading during check/score fetch
    console.log("[Debug] checkGameCompletion - isLoading is now true.");

    try {
        console.log(`[Debug] checkGameCompletion - Calling api.isGameCompleted for gameId: ${gameId.value}`);
        const response = await api.isGameCompleted({ gameId: gameId.value });
        console.log("[Debug] checkGameCompletion - /game/completed response:", response.data);

        if (response.data === true) {
            console.log("[Debug] checkGameCompletion - Game confirmed complete by backend.");
            isCompleted.value = true;
            stopTimer();
            console.log("[Debug] checkGameCompletion - Timer stopped. Fetching final score...");
            try {
                const scoreResponse = await api.getScore({ gameId: gameId.value });
                console.log("[Debug] checkGameCompletion - /game/score response:", scoreResponse.data);
                finalScore.value = scoreResponse.data;
                console.log("[Debug] checkGameCompletion - Final score set. Showing modal...");
                showGameEndModal.value = true; // Show the modal
            } catch (scoreErr: any) {
                console.error("[Debug] checkGameCompletion - Error fetching score:", scoreErr);
                alert(`Could not fetch final score: ${scoreErr.response?.data?.message || scoreErr.message || 'Unknown error'}`);
                 // Maybe show modal anyway with score 'N/A'?
                 finalScore.value = null; // Ensure score is null if fetch failed
                 showGameEndModal.value = true;
            }
        } else {
            console.log("[Debug] checkGameCompletion - Backend says game is NOT complete yet.");
             // Optional: Add feedback if grid is full but incorrect?
             // alert("Grid is full, but some values are incorrect!");
        }
    } catch (err: any) {
        console.error("[Debug] checkGameCompletion - Error during API call or processing:", err);
        alert(`Error checking completion: ${err.response?.data?.message || err.message || 'Unknown error'}`);
    } finally {
        console.log("[Debug] checkGameCompletion - Entering finally block.");
        isLoading.value = false; // Reset loading state
        console.log("[Debug] checkGameCompletion - isLoading set to false.");
    }
};

// Submit Score to Leaderboard
const submitScore = async (username: string): Promise<void> => {
    if (!username || isLoading.value || finalScore.value === null || !gameId.value) return;

    isLoading.value = true;
    console.log("[Debug] submitScore - Submitting score...");
    try {
        await api.addLeaderboardRecord({
            username,
            score: finalScore.value,
            difficulty: difficulty.value
         });
        showGameEndModal.value = false; // Close modal on success
        await fetchLeaderboard(); // Refresh leaderboard
        alert(`Score submitted for ${username}!`);
        resetGameState(); // Reset game after successful submission

    } catch (err: any) {
        console.error("[Debug] submitScore - Failed to submit score:", err);
        alert(`Error submitting score: ${err.response?.data?.message || err.message || 'Unknown error'}`);
    } finally {
        isLoading.value = false;
        console.log("[Debug] submitScore - Finished score submission.");
    }
};

// Timer Functions
const startTimer = (): void => {
    stopTimer(); // Clear any existing interval
    if (!gameStartTime.value) gameStartTime.value = Date.now(); // Fallback if start time missing
    timerInterval.value = window.setInterval(() => { // Use window.setInterval for clarity on type
        if (!isPaused.value && !isCompleted.value && gameStartTime.value) {
            elapsedTime.value = Math.floor((Date.now() - gameStartTime.value) / 1000);
        }
    }, 1000);
};
const stopTimer = (): void => {
    if (timerInterval.value !== null) {
        clearInterval(timerInterval.value);
        timerInterval.value = null;
    }
};
const togglePause = (): void => {
    if (!isCompleted.value) { // Can't pause completed game
         isPaused.value = !isPaused.value;
    }
};
const formatTime = (totalSeconds: number): string => {
    const minutes = Math.floor(totalSeconds / 60);
    const seconds = totalSeconds % 60;
    return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
};

// Keyboard Event Handler
const handleKeyDown = (event: KeyboardEvent): void => {
    if (showGameEndModal.value || isLoading.value) return;
    if (!gameInProgress.value || isPaused.value || isCompleted.value) return;

    const { key } = event;

    if (selectedCell.value) {
        const { row, col } = selectedCell.value;
        const cellKey = `${row}-${col}`;
        const isEditable = initialGrid.value[row]?.[col] === 0 && !hintCells.has(cellKey);

        if (isEditable) {
            if (key >= '1' && key <= '9') {
                const digit = parseInt(key);
                if (digitCounts.value[digit] >= 9) {
                    console.log(`Cannot input digit ${digit}: All instances are already on the grid.`);
                    event.preventDefault(); return;
                }
                event.preventDefault(); handleNumberInput(digit); return;
            } else if (key === 'Backspace' || key === 'Delete') {
                event.preventDefault(); clearSelectedCell(); return;
            }
        }

        // Arrow Key Navigation
        let newRow = row; let newCol = col; let moved = false;
        if (key === 'ArrowUp') { newRow = Math.max(0, row - 1); moved = true; }
        else if (key === 'ArrowDown') { newRow = Math.min(gridDimension.value - 1, row + 1); moved = true; }
        else if (key === 'ArrowLeft') { newCol = Math.max(0, col - 1); moved = true; }
        else if (key === 'ArrowRight') { newCol = Math.min(gridDimension.value - 1, col + 1); moved = true; }

        if (moved) {
            event.preventDefault(); handleCellSelect({ row: newRow, col: newCol }); return;
        }
    }
     // --- Add global key listeners (e.g., 'P' for pause) ---
     if (key.toUpperCase() === 'P') {
        event.preventDefault();
        togglePause();
     }
     // Add 'H' for hint?
     if (key.toUpperCase() === 'H') {
        event.preventDefault();
        requestHint();
     }
};

// --- Lifecycle Hooks ---
onMounted(() => {
    fetchLeaderboard();
    window.addEventListener('keydown', handleKeyDown);
});

onUnmounted(() => {
    stopTimer();
    window.removeEventListener('keydown', handleKeyDown);
});

</script>

<template>
  <div id="app-container">
    <header class="app-header">
      <h1>Vue Sudoku TS</h1>
      <Controls
        :current-difficulty="difficulty"
        :errors="errors"
        :elapsed-time="elapsedTime"
        :hints-remaining="hintsRemaining"
        :is-paused="isPaused"
        :game-in-progress="gameInProgress"
        :is-loading="isLoading"
        @start-game="startGame"
        @use-hint="requestHint"
        @toggle-pause="togglePause"
      ></Controls>
    </header>

    <main class="main-content">
      <div class="game-area">
        <SudokuGrid
          v-if="gameId"
          :grid-data="grid"
          :initial-grid="initialGrid"
          :selected-cell="selectedCell"
          :error-cells="errorCells"
          :hint-cells="hintCells"
          :is-paused="isPaused"
          @cell-selected="handleCellSelect"
        ></SudokuGrid>
        <div v-else class="start-prompt">
          <p>Select a difficulty and press "Start Game" to begin.</p>
           <div v-if="isLoading">Loading...</div> <!-- Only show loading if no gameId AND loading -->
        </div>

        <!-- Show number pad only during active game -->
        <NumberPad
           v-if="gameInProgress"
           :digit-counts="digitCounts"
           :is-paused="isPaused"
        ></NumberPad>
      </div>

      <aside class="sidebar">
         <!-- Show loading indicator for leaderboard if loading AND no data exists yet -->
         <Leaderboard
            :leaderboard-data="leaderboardData"
            :is-loading="isLoading && leaderboardData === null"
         ></Leaderboard>
      </aside>
    </main>

    <GameEndModal
      v-if="showGameEndModal"
      :score="finalScore"
      :difficulty="difficulty"
      :time="formatTime(elapsedTime)"
      @submit-score="submitScore"
      @close="showGameEndModal = false; /* Don't reset game on close, only on submit or New Game */"
    ></GameEndModal>

    <!-- Optional Global Loading Overlay -->
    <div v-if="isLoading" class="loading-overlay"><span>Loading...</span></div>
  </div>
</template>

<!-- Global Styles -->
<style>
 :root {
   --border-color: #bbb;
   --strong-border-color: #333;
   --main-bg: #f8f8f8;
   --cell-bg: #fff;
   --cell-prefilled-bg: #e0e0e0;
   --cell-selected-bg: #fffbaf; /* Yellowish selection */
   --cell-error-bg: #fff;
   --cell-error-text: #dc3545;
   --cell-hint-bg: #d1ecf1; /* Light blue for hint bg */
   --cell-hint-text: #0c5460;
   --button-bg: #e9e9e9;
   --button-hover-bg: #dcdcdc;
   --button-disabled-bg: #f5f5f5;
   --button-disabled-text: #aaa;
   --header-bg: #f0f0f0;
   --sidebar-bg: #f5f5f5;
   --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
   --color-success: #28a745;
   --color-win-animation: #4CAF50;
 }

 body {
   font-family: var(--font-family);
   margin: 0;
   background-color: var(--main-bg);
   color: #333;
   line-height: 1.5;
 }

 *, *::before, *::after { box-sizing: border-box; }

 #app-container {
   display: flex;
   flex-direction: column;
   max-width: 1000px;
   min-height: 100vh;
   margin: 0 auto;
   background: #fff;
   box-shadow: 0 2px 5px rgba(0,0,0,0.1);
 }

 .app-header {
    background-color: var(--header-bg);
    padding: 1em;
    border-bottom: 1px solid var(--border-color);
    text-align: center;
 }
  .app-header h1 { margin: 0 0 0.5em 0; font-weight: 600; }

 .main-content {
   display: flex;
   flex-direction: row;
   padding: 1em;
   gap: 1.5em;
   flex-grow: 1;
 }

 .game-area {
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1em;
 }

 .start-prompt {
     margin-top: 50px;
     padding: 20px;
     text-align: center;
     color: #555;
     font-size: 1.1em;
 }

 .loading-overlay {
     position: fixed; inset: 0;
     background-color: rgba(255, 255, 255, 0.8); /* Slightly more opaque */
     display: flex; justify-content: center; align-items: center;
     font-size: 1.5em; color: #333; z-index: 1000;
 }
 .loading-overlay span { /* Style text inside overlay */
    background-color: #fff; padding: 15px 25px; border-radius: 5px; box-shadow: 0 2px 4px rgba(0,0,0,0.2);
 }


 .sidebar {
    flex-basis: 280px; flex-shrink: 0;
    background-color: var(--sidebar-bg); padding: 1em;
    border-left: 1px solid var(--border-color);
 }

 @media (max-width: 800px) {
     .main-content { flex-direction: column; }
     .sidebar { flex-basis: auto; width: 100%; border-left: none; border-top: 1px solid var(--border-color); padding: 0.5em; }
     .game-area { width: 100%; }
 }

 button {
    padding: 8px 15px; border: 1px solid var(--border-color); border-radius: 4px;
    background-color: var(--button-bg); cursor: pointer; font-size: 0.95rem;
    transition: background-color 0.2s ease;
 }
  button:hover:not(:disabled) { background-color: var(--button-hover-bg); }
  button:disabled { background-color: var(--button-disabled-bg); color: var(--button-disabled-text); cursor: not-allowed; opacity: 0.7; }
</style>