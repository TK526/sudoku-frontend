<script setup>
import { computed } from 'vue';

const props = defineProps({
    digitCounts: { type: Object, required: true },
    isPaused: { type: Boolean, default: false }
});

// No longer emitting number-selected
// const emit = defineEmits(['number-selected']);

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9];

const isDigitDisabled = (num) => {
    // Disable if count is 9 or more. Pause doesn't affect display.
    return (props.digitCounts[num] !== undefined && props.digitCounts[num] >= 9);
};

// Removed selectNumber function
// const selectNumber = (num) => {
//     if (!isDigitDisabled(num)) {
//         emit('number-selected', num);
//     }
// };
</script>

<template>
  <div class="number-pad">
     <p>Available Digits:</p>
    <button
      v-for="num in numbers"
      :key="num"
      :disabled="isDigitDisabled(num)"
      class="digit-display-button" 
      tabindex="-1" 
    >
      {{ num }}
    </button>
  </div>
</template>

<style scoped>
.number-pad {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 5px;
  margin-top: 1em;
   flex-wrap: wrap;
}

 .number-pad p {
    margin-right: 10px;
    font-size: 0.9rem;
    color: #555;
 }

/* Style for the buttons that now only display digits */
.digit-display-button {
  width: 40px;
  height: 40px;
  font-size: 1.2em;
  border: 1px solid var(--border-color);
  background-color: var(--button-bg);
  border-radius: 4px;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #333; /* Default color */
  cursor: default; /* No longer clickable */
}

.digit-display-button:disabled {
   background-color: var(--button-disabled-bg);
   color: var(--button-disabled-text);
   opacity: 0.7;
   /* Optional: Add line-through */
   /* text-decoration: line-through; */
}
</style>