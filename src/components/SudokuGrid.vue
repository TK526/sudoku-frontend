<script setup lang="ts">
import { computed } from 'vue';
import type { CellPosition } from '../types'; // Import shared type

// Define Props
interface Props {
  gridData: number[][];
  initialGrid: number[][]; // To know which cells were pre-filled
  selectedCell: CellPosition | null;
  errorCells: Set<string>; // Reactive Set passed down ('row-col')
  hintCells: Set<string>; // Reactive Set passed down ('row-col')
  isPaused: boolean;
  // Props for win animation - Need to test TODO
  // winningRow?: number | null;
  // winningCol?: number | null;
  // winningBox?: { startRow: number, startCol: number } | null;
}
const props = defineProps<Props>();

// Define Emits
const emit = defineEmits<{
  (event: 'cell-selected', position: CellPosition): void;
}>();

// Computed properties
const gridDimension = computed(() => props.gridData?.length || 9);
const subgridSize = computed(() => Math.sqrt(gridDimension.value)); // Assumes perfect square

// Helper functions for cell state
const isPrefilled = (row: number, col: number): boolean => {
  return !!props.initialGrid[row]?.[col]; // Check if non-zero in initial grid
};

const isSelected = (row: number, col: number): boolean => {
  return props.selectedCell?.row === row && props.selectedCell?.col === col;
};

const isError = (row: number, col: number): boolean => {
  return props.errorCells.has(`${row}-${col}`);
};

const isHint = (row: number, col: number): boolean => {
  return props.hintCells.has(`${row}-${col}`);
};

// Dynamically calculate CSS classes for a cell
const getCellClass = (row: number, col: number): string[] => {
  const classes = ['sudoku-cell'];
  const prefilled = isPrefilled(row, col);
  const hint = isHint(row, col);

  if (prefilled) classes.push('prefilled');
  if (isSelected(row, col)) classes.push('selected');
  if (isError(row, col)) classes.push('error');
  if (hint) classes.push('hint'); // Hints override user input style
  if (!prefilled && !hint) classes.push('user-input'); // Style user numbers differently

  // Add thicker borders for subgrids
  if ((col + 1) % subgridSize.value === 0 && col < gridDimension.value - 1) {
    classes.push('thick-border-right');
  }
  if ((row + 1) % subgridSize.value === 0 && row < gridDimension.value - 1) {
    classes.push('thick-border-bottom');
  }

  // Add win animation - Need to test
  // if (props.winningRow === row) classes.push('win-highlight');
  // if (props.winningCol === col) classes.push('win-highlight');
  // if (props.winningBox && row >= props.winningBox.startRow && row < props.winningBox.startRow + subgridSize.value && col >= props.winningBox.startCol && col < props.winningBox.startCol + subgridSize.value) classes.push('win-highlight');

  return classes;
};

// Handler for clicking a cell
const selectCell = (row: number, col: number): void => {
  // Prevent selection when paused or if cell is pre-filled
  if (props.isPaused || isPrefilled(row, col)) {
     // Optionally deselect if clicking prefilled?
     // emit('cell-selected', null); // Or just do nothing
      return;
  }
  emit('cell-selected', { row, col });
};

</script>

<template>
  <div class="sudoku-grid-container" :class="{ paused: isPaused }">
    <!-- The main grid element -->
    <div v-if="gridData.length > 0" class="sudoku-grid" :style="{ '--grid-size': gridDimension }">
      <!-- Loop through rows FIRST -->
      <template v-for="(row, rowIndex) in gridData" :key="`row-${rowIndex}`">
        <!-- Loop through columns SECOND, generating the cells directly -->
        <div
          v-for="(cellValue, colIndex) in row"
          :key="`cell-${rowIndex}-${colIndex}`"
          :class="getCellClass(rowIndex, colIndex)"
          @click="selectCell(rowIndex, colIndex)"
          role="button"
          :aria-label="`Cell Row ${rowIndex+1} Column ${colIndex+1} Value ${cellValue || 'Empty'}`"
          :aria-selected="isSelected(rowIndex, colIndex)"
          :aria-disabled="isPrefilled(rowIndex, colIndex)"
          tabindex="0" <!-- Make cells focusable for accessibility -->
          @keydown.enter.space="selectCell(rowIndex, colIndex)" <!-- Allow selection with keyboard -->
        >
          {{ cellValue === 0 ? '' : cellValue }}
        </div>
      </template>
    </div>
    <div v-else class="loading-grid">Loading Grid...</div>
    <div v-if="isPaused" class="paused-overlay">Paused</div>
  </div>
</template>

<style scoped>
 .sudoku-grid-container {
    position: relative;
    width: 100%;
    max-width: 540px; /* Or adjust as needed */
    aspect-ratio: 1 / 1;
    margin-left: auto;
    margin-right: auto;
 }

 .loading-grid {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
    font-style: italic;
    color: #777;
 }

 .sudoku-grid {
   display: grid;
   grid-template-columns: repeat(var(--grid-size, 9), 1fr);
   grid-template-rows: repeat(var(--grid-size, 9), 1fr);
   border: 3px solid var(--strong-border-color, #333);
   width: 100%;
   height: 100%;
   background-color: var(--cell-bg, #fff);
 }

 .sudoku-cell {
   display: flex;
   justify-content: center;
   align-items: center;
   font-size: clamp(1.5rem, 5vw, 2rem); /* Slightly adjusted */
   border: 1px solid var(--border-color, #bbb);
   cursor: pointer;
   user-select: none;
   transition: background-color 0.15s ease-out;
   background-color: var(--cell-bg, #fff);
   color: #111;
   box-sizing: border-box;
   position: relative; /* For potential future animations/overlays */
 }

/* Add focus outline for accessibility */
.sudoku-cell:focus {
    outline: 2px solid blue;
    outline-offset: -2px;
    z-index: 2;
}


 .sudoku-cell.user-input { /* Style for user-entered numbers */
     font-weight: 600; /* Bold */
     color: #0056b3; /* Blue */
 }
 .sudoku-cell.user-input:hover {
     background-color: #f0f8ff; /* Light hover */
 }

 .sudoku-cell.prefilled {
   background-color: var(--cell-prefilled-bg, #e0e0e0);
   cursor: default;
   font-weight: 600; /* Make bold */
   color: #222; /* Darker */
 }

 .sudoku-cell.selected {
   background-color: var(--cell-selected-bg, #fffbaf);
   /* outline: 2px solid #e0a800; Optional border */
   z-index: 1;
 }

 .sudoku-cell.error {
    background-color: var(--cell-error-bg, #fff);
    color: var(--cell-error-text, #dc3545); /* Red text */
    /* Add back subtle background if preferred */
    /* background-color: #f8d7da; */
    animation: shake 0.4s ease-in-out;
    font-weight: 700; /* Extra bold error */
 }

 .sudoku-cell.hint {
    background-color: var(--cell-hint-bg, #d1ecf1);
    color: var(--cell-hint-text, #0c5460);
    font-style: italic;
    font-weight: 600; /* Make hints stand out */
    cursor: default; /* Hints usually aren't re-selectable once placed */
 }

 .thick-border-right {
   border-right: 2px solid var(--strong-border-color, #333);
 }
 .thick-border-bottom {
   border-bottom: 2px solid var(--strong-border-color, #333);
 }

  .paused-overlay {
     position: absolute;
     inset: 0; /* Covers the whole container */
     background-color: rgba(180, 180, 180, 0.65);
     display: flex;
     justify-content: center;
     align-items: center;
     font-size: 2.5em;
     color: white;
     font-weight: bold;
     text-shadow: 1px 1px 2px black;
     z-index: 10;
     pointer-events: none;
     border-radius: inherit;
  }

 @keyframes shake {
   0%, 100% { transform: translateX(0); }
   25%, 75% { transform: translateX(-4px); }
   50% { transform: translateX(4px); }
 }

 /* Win animation style (Example) */
 .win-highlight {
    animation: winPulse 0.8s ease-in-out infinite alternate;
 }

 @keyframes winPulse {
    from { background-color: var(--cell-bg); }
    to { background-color: var(--color-win-animation); color: white; }
 }
</style>