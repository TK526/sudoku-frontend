<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
    currentDifficulty: String,
    score: Number, // Maybe replace with errors?
    errors: Number,
    elapsedTime: Number,
    hintsRemaining: Number,
    isPaused: Boolean,
    gameInProgress: Boolean,
    isLoading: Boolean,
});

const emit = defineEmits(['start-game', 'use-hint', 'toggle-pause']);

const selectedDifficulty = ref(props.currentDifficulty || 'beginner');
const difficulties = ['beginner', 'intermediate', 'hard', 'expert'];

const formatTime = (totalSeconds) => {
    const minutes = Math.floor(totalSeconds / 60);
    const seconds = totalSeconds % 60;
    return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
};

const handleStart = () => {
    if (!props.isLoading) {
        emit('start-game', selectedDifficulty.value);
    }
};

const handleHint = () => {
     if (!props.isLoading && !props.isPaused && props.hintsRemaining > 0) {
        emit('use-hint');
    }
};

const handlePause = () => {
    if (props.gameInProgress) {
        emit('toggle-pause');
    }
};

</script>

<template>
  <div class="controls-container">
    <div class="difficulty-selector" v-if="!gameInProgress">
      <label for="difficulty">Difficulty: </label>
      <select id="difficulty" v-model="selectedDifficulty" :disabled="isLoading">
        <option v-for="diff in difficulties" :key="diff" :value="diff">{{ diff }}</option>
      </select>
       <button @click="handleStart" :disabled="isLoading">
        {{ isLoading ? 'Starting...' : 'Start Game' }}
        </button>
    </div>

    <div v-if="gameInProgress" class="game-info">
      <span>Level: {{ currentDifficulty }}</span>
      <!-- <span>Score: {{ score }}</span> -->
       <span>Errors: {{ errors }}</span>
      <span>Time: {{ formatTime(elapsedTime) }}</span>
      <button @click="handlePause" :disabled="isLoading">{{ isPaused ? 'Resume' : 'Pause' }}</button>
      <button @click="handleHint" :disabled="isLoading || isPaused || hintsRemaining <= 0">
        💡 Hint ({{ hintsRemaining }})
      </button>
       <button @click="handleStart" :disabled="isLoading">New Game</button> <!-- Allow starting new game -->
    </div>
  </div>
</template>

<style scoped>
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