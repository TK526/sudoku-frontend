<script setup lang="ts">
// Define Props
interface Props {
  // Object where keys are digits (1-9) and values are their counts
  digitCounts: { [key: number]: number };
  isPaused: boolean; // Although not interactive, pause might affect appearance slightly
}
const props = defineProps<Props>();

// No emits needed anymore

const numbers: number[] = [1, 2, 3, 4, 5, 6, 7, 8, 9];

// Determine if a digit button should be visually 'disabled' (greyed out)
const isDigitComplete = (num: number): boolean => {
  return props.digitCounts[num] !== undefined && props.digitCounts[num] >= 9;
};

</script>

<template>
  <div class="number-pad">
     <p class="pad-label">Available Digits:</p>
    <div class="pad-buttons">
        <button
          v-for="num in numbers"
          :key="num"
          :disabled="isDigitComplete(num)"
          class="digit-display-button"
          tabindex="-1" <!-- Not interactive -->
        >
          {{ num }}
        </button>
    </div>
  </div>
</template>

<style scoped>
.number-pad {
  display: flex;
  flex-direction: column; /* Stack label and buttons */
  align-items: center; /* Center items */
  gap: 8px; /* Space between label and buttons */
  margin-top: 1em;
  width: 100%;
  max-width: 300px; /* Limit width */
}
.pad-label {
    margin: 0;
    font-size: 0.9rem;
    color: #555;
    font-weight: 500;
}
.pad-buttons {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 5px;
    flex-wrap: wrap;
}

.digit-display-button {
  width: 38px; /* Slightly smaller */
  height: 38px;
  font-size: 1.1em;
  font-weight: 500;
  border: 1px solid var(--border-color);
  background-color: var(--button-bg);
  border-radius: 4px;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #333;
  cursor: default; /* Not interactive */
   transition: background-color 0.2s, color 0.2s;
}

.digit-display-button:disabled {
   background-color: var(--button-disabled-bg);
   color: var(--button-disabled-text);
   opacity: 0.7;
   /* text-decoration: line-through; Optional */
}
</style>