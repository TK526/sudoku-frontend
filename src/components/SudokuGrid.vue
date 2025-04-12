<script setup>
import { computed } from 'vue';

const props = defineProps({
  gridData: { type: Array, required: true },
  initialGrid: { type: Array, required: true },
  selectedCell: { type: Object, default: null },
  errorCells: { type: Object, required: true }, // Reactive Set
  hintCells: { type: Object, required: true }, // Reactive Set
  isPaused: { type: Boolean, default: false }
});

const emit = defineEmits(['cell-selected']);

const gridDimension = computed(() => props.gridData.length);
const subgridSize = computed(() => Math.sqrt(gridDimension.value));

const isPrefilled = (row, col) => {
    // Check the initial grid state passed down
    return props.initialGrid[row]?.[col] !== 0;
};

 const isSelected = (row, col) => {
    return props.selectedCell?.row === row && props.selectedCell?.col === col;
 };

 const isError = (row, col) => {
    return props.errorCells.has(`${row}-${col}`);
 };

  const isHint = (row, col) => {
    return props.hintCells.has(`${row}-${col}`);
 };


const getCellClass = (row, col) => {
  const classes = ['sudoku-cell'];
  if (isPrefilled(row, col)) classes.push('prefilled');
  if (isSelected(row, col)) classes.push('selected');
  if (isError(row, col)) classes.push('error');
  if (isHint(row, col)) classes.push('hint'); // Added hint class

  // Add thicker borders for subgrids
  if ((col + 1) % subgridSize.value === 0 && col < gridDimension.value - 1) {
    classes.push('thick-border-right');
  }
  if ((row + 1) % subgridSize.value === 0 && row < gridDimension.value - 1) {
    classes.push('thick-border-bottom');
  }

  return classes;
};

const selectCell = (row, col) => {
    if (props.isPaused) return; // Prevent selection when paused
    // Emit only if the cell is NOT prefilled
    // Let App.vue handle the logic of whether to actually select it
     emit('cell-selected', { row, col });

};
</script>

<template>
    <div class="sudoku-grid-container" :class="{ paused: isPaused }">
      <!-- The main grid container - direct children will be the cells -->
      <div v-if="gridData.length > 0" class="sudoku-grid" :style="{ '--grid-size': gridDimension }">
        <!-- Loop through rows FIRST -->
        <template v-for="(row, rowIndex) in gridData" :key="`row-${rowIndex}`">
          <!-- Loop through columns SECOND, generating the cells directly -->
          <div
            v-for="(cellValue, colIndex) in row"
            :key="`cell-${rowIndex}-${colIndex}`"
            :class="getCellClass(rowIndex, colIndex)"
            @click="selectCell(rowIndex, colIndex)"
          >
            {{ cellValue === 0 ? '' : cellValue }}
          </div>
        </template>
      </div>
      <div v-else>Loading Grid...</div>
      <div v-if="isPaused" class="paused-overlay">Paused</div>
    </div>
</template>

<style scoped>
 .sudoku-grid-container {
    position: relative;
    margin-bottom: 1em;
    width: 100%;
    max-width: 540px; /* Adjust size as needed, keep divisible by 9 */
    aspect-ratio: 1 / 1;
    margin-left: auto;
    margin-right: auto;
 }

/* This is the main grid element */
.sudoku-grid {
  display: grid;
  /* Define 9 equal columns and rows */
  grid-template-columns: repeat(var(--grid-size, 9), 1fr);
  grid-template-rows: repeat(var(--grid-size, 9), 1fr);
  /* Use the strong border for the outer edge */
  border: 3px solid var(--strong-border-color, #333); /* Added fallback */
  width: 100%;
  height: 100%;
  background-color: var(--cell-bg, #fff); /* Added fallback */
}

/* Styles for each individual cell */
.sudoku-cell {
  display: flex;
  justify-content: center;
  align-items: center;
  /* Adjust font size as needed */
  font-size: clamp(1.5rem, 5vw, 2.2rem);
  /* Use the standard border color for thin internal lines */
  border: 1px solid var(--border-color, #bbb); /* Added fallback */
  cursor: pointer;
  user-select: none;
  transition: background-color 0.2s ease;
  background-color: var(--cell-bg, #fff);
  color: #111; /* Default text color */
  box-sizing: border-box; /* Ensure border doesn't add to size */
}

/* Style for user-editable cells (not prefilled) */
.sudoku-cell:not(.prefilled) {
    font-weight: bold;
    color: #0056b3; /* Blue color for user input */
}
.sudoku-cell:not(.prefilled):hover {
    background-color: #f0f8ff; /* Light hover */
}

/* Style for the initial numbers given */
.sudoku-cell.prefilled {
  background-color: var(--cell-prefilled-bg, #e0e0e0); /* Added fallback */
  cursor: default;
  font-weight: bold;
  color: #333; /* Darker color */
}

/* Style for the currently selected cell (yellow) */
.sudoku-cell.selected {
  background-color: var(--cell-selected-bg, #fffbaf); /* Added fallback yellow */
  /* Optional: Add outline */
  /* outline: 2px solid #f0c000; */
  z-index: 1; /* Ensure it's visually on top if needed */
}

/* Style for cells marked as errors */
.sudoku-cell.error {
   background-color: var(--cell-error-bg, #fff); /* Keep background white (or change if needed) */
   color: var(--cell-error-text, #dc3545); /* Red text for error */
   animation: shake 0.3s;
   font-weight: bold; /* Ensure error text is bold */
}

/* Style for cells revealed by hints */
.sudoku-cell.hint {
   background-color: var(--cell-hint-bg, #d1ecf1); /* Light blue */
   color: var(--cell-hint-text, #0c5460); /* Darker blue text */
   font-style: italic;
   font-weight: bold;
}

/* Thicker borders for subgrids (3x3 boxes) */
.thick-border-right {
  border-right: 2px solid var(--strong-border-color, #333);
}
.thick-border-bottom {
  border-bottom: 2px solid var(--strong-border-color, #333);
}

 .paused-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(128, 128, 128, 0.6);
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 2em;
    color: white;
    font-weight: bold;
    z-index: 10;
    pointer-events: none;
 }

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-3px); }
  75% { transform: translateX(3px); }
}
</style>