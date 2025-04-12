<!-- ./components/Controls.vue -->
<script setup lang="ts"> // Added lang="ts"
import { ref } from 'vue';

// Define component props using TypeScript interface
interface Props {
  currentDifficulty: string;
  // score: number; // Score prop is not used effectively, remove or keep as 0? Keeping for now.
  errors: number;
  elapsedTime: number;
  hintsRemaining: number;
  isPaused: boolean;
  gameInProgress: boolean;
  isLoading: boolean;
}
// Use withDefaults if needed, but here all seem required or have internal defaults
const props = defineProps<Props>();

// Define component emits using TypeScript syntax
const emit = defineEmits<{
  (event: 'startGame', difficulty: string): void;
  (event: 'useHint'): void;
  (event: 'togglePause'): void;
}>();

// Local state for difficulty selection before game starts
const selectedDifficulty = ref<string>(props.currentDifficulty || 'beginner');
const difficulties: string[] = ['beginner', 'intermediate', 'hard', 'expert'];

// Format time function with types
const formatTime = (totalSeconds: number): string => {
    const minutes = Math.floor(totalSeconds / 60);
    const seconds = totalSeconds % 60;
    return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
};

// Event handlers
const handleStart = (): void => {
    if (!props.isLoading) {
        emit('startGame', selectedDifficulty.value);
    }
};

const handleHint = (): void => {
     if (!props.isLoading && props.gameInProgress && !props.isPaused && props.hintsRemaining > 0) {
        emit('useHint');
    }
};

const handlePause = (): void => {
    // Allow toggling pause even if loading? Usually yes.
    if (props.gameInProgress) {
        emit('togglePause');
    }
};

</script>

<template>
  <div class="controls-container">
    <div class="difficulty-selector" v-if="!gameInProgress">
      <label for="difficulty">Difficulty: </label>
      <select id="difficulty" v-model="selectedDifficulty" :disabled="isLoading">
        <!-- Capitalize difficulty names for display -->
        <option v-for="diff in difficulties" :key="diff" :value="diff">
            {{ diff.charAt(0).toUpperCase() + diff.slice(1) }}
        </option>
      </select>
       <button @click="handleStart" :disabled="isLoading">
        {{ isLoading ? 'Starting...' : 'Start Game' }}
        </button>
    </div>

    <div v-if="gameInProgress" class="game-info">
      <span>Level: {{ currentDifficulty }}</span>
      <!-- Removed score display as it wasn't implemented -->
       <span>Errors: {{ errors }}</span>
      <span>Time: {{ formatTime(elapsedTime) }}</span>
      <button @click="handlePause" :disabled="isLoading">{{ isPaused ? 'Resume' : 'Pause' }}</button>
      <button @click="handleHint" :disabled="isLoading || isPaused || hintsRemaining <= 0">
        💡 Hint ({{ hintsRemaining }})
      </button>
       <button @click="handleStart" :disabled="isLoading">New Game</button>
    </div>
  </div>
</template>

<style scoped>
/* Styles remain the same as provided in the dump */
.controls-container {
  display: flex;
  justify-content: center; /* Center items */
  align-items: center;
  gap: 15px; /* Space between elements */
  flex-wrap: wrap; /* Allow wrapping on smaller screens */
   padding: 0.5em 0;
}

.difficulty-selector, .game-info {
    display: flex;
    align-items: center;
    gap: 10px;
}

 label {
    font-weight: bold;
 }

 select, button {
    padding: 8px 12px;
    border: 1px solid var(--border-color);
    border-radius: 4px;
    background-color: var(--button-bg);
    cursor: pointer;
    font-size: 0.9rem;
 }

 select {
    min-width: 120px;
 }

 button:hover:not(:disabled) {
    background-color: var(--button-hover-bg);
 }

 button:disabled {
    background-color: var(--button-disabled-bg);
    color: var(--button-disabled-text);
    cursor: not-allowed;
 }

 span {
    font-size: 0.9rem;
    background-color: #fff;
    padding: 5px 8px;
    border-radius: 4px;
    border: 1px solid var(--border-color);
 }
</style>