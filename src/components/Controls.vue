<script setup lang="ts">
import { ref, computed } from 'vue';

// Define component props
interface Props {
  currentDifficulty: string;
  errors: number;
  elapsedTime: number;
  hintsRemaining: number;
  isPaused: boolean;
  gameInProgress: boolean;
  isLoading: boolean; // To disable buttons during API calls - kind of like a rate limit
}
const props = defineProps<Props>();

// Define component emits
const emit = defineEmits<{
  (event: 'startGame', difficulty: string): void;
  (event: 'useHint'): void;
  (event: 'togglePause'): void;
}>();

// Local state for difficulty selection before game starts
const selectedDifficulty = ref<string>(props.currentDifficulty || 'beginner');
const difficulties: string[] = ['beginner', 'intermediate', 'hard', 'expert'];

// Computed property to format time
const formattedTime = computed(() => {
  const minutes = Math.floor(props.elapsedTime / 60);
  const seconds = props.elapsedTime % 60;
  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
});

// Event handlers
const handleStart = () => {
  if (!props.isLoading) {
    emit('startGame', selectedDifficulty.value);
  }
};

const handleHint = () => {
  // Check all conditions before emitting
  if (!props.isLoading && props.gameInProgress && !props.isPaused && props.hintsRemaining > 0) {
    emit('useHint');
  }
};

const handlePause = () => {
  if (props.gameInProgress && !props.isLoading) { // Prevent pausing during critical load.
    emit('togglePause');
  }
};

</script>

<template>
  <div class="controls-container">
    <!-- Difficulty Selection & Start (shown when game not in progress) -->
    <div class="difficulty-selector" v-if="!gameInProgress">
      <label for="difficulty">Difficulty: </label>
      <select id="difficulty" v-model="selectedDifficulty" :disabled="isLoading">
        <option v-for="diff in difficulties" :key="diff" :value="diff">
          {{ diff.charAt(0).toUpperCase() + diff.slice(1) }} <!-- Capitalize -->
        </option>
      </select>
      <button @click="handleStart" :disabled="isLoading">
        {{ isLoading ? 'Starting...' : 'Start Game' }}
      </button>
    </div>

    <!-- Game Info & Actions (shown during game) -->
    <div v-if="gameInProgress" class="game-info">
      <span>Level: {{ currentDifficulty }}</span>
      <span>Errors: {{ errors }}</span>
      <span>Time: {{ formattedTime }}</span>
      <button @click="handlePause" :disabled="isLoading">
        {{ isPaused ? 'Resume' : 'Pause' }} <span v-if="!isPaused">❚❚</span><span v-else>▶</span>
      </button>
      <button
        @click="handleHint"
        :disabled="isLoading || isPaused || hintsRemaining <= 0"
        class="hint-button"
        title="Use a Hint"
      >
        💡 Hint ({{ hintsRemaining }}) <!-- Hint Icon -->
      </button>
      <button @click="handleStart" :disabled="isLoading">
        New Game
      </button> <!-- Allow starting new game -->
    </div>
  </div>
</template>

<style scoped>
.controls-container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  flex-wrap: wrap;
  padding: 0.5em 0;
  min-height: 40px; /* Ensure consistent height */
}

.difficulty-selector, .game-info {
    display: flex;
    align-items: center;
    gap: 10px; /* Spacing between items */
}

label {
    font-weight: 500; /* Slightly less bold */
}

select {
    padding: 7px 10px;
    border: 1px solid var(--border-color);
    border-radius: 4px;
    background-color: #fff;
    min-width: 120px;
    cursor: pointer;
}

span { /* Style for info displays */
    font-size: 0.9rem;
    background-color: #fff;
    padding: 5px 10px;
    border-radius: 4px;
    border: 1px solid var(--border-color);
    min-width: 60px; /* Give some minimum width */
    text-align: center;
}

.hint-button span {
    background: none; /* Remove background for icon */
    border: none;
    padding: 0;
    min-width: auto;
}

button span { /* Style for pause/play icon */
   font-size: 0.8em;
   margin-left: 4px;
}
</style>