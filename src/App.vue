<script setup lang="ts">
// Imports will go here later
</script>

<template>
  <div id="app-container">
    <header class="app-header">
      <h1>Vue Sudoku TS</h1>
      <!-- Controls Component will go here -->
    </header>

    <main class="main-content">
      <div class="game-area">
        <!-- SudokuGrid Component will go here -->
        <!-- NumberPad Component will go here -->
         <div class="start-prompt">
           <p>Select a difficulty and press "Start Game" to begin.</p>
           <!-- Loading indicator here later -->
         </div>
      </div>

      <aside class="sidebar">
        <!-- Leaderboard Component will go here -->
      </aside>
    </main>

    <!-- GameEndModal Component will go here -->
    <!-- Global Loading Overlay here later -->
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
   --color-win-animation: #4CAF50; /* Green for win animation */
 }

 body {
   font-family: var(--font-family);
   margin: 0;
   background-color: var(--main-bg);
   color: #333;
   line-height: 1.5;
 }

 *, *::before, *::after {
    box-sizing: border-box;
 }

 #app-container {
   display: flex;
   flex-direction: column;
   max-width: 1000px;
   min-height: 100vh; /* Ensure it takes full height */
   margin: 0 auto; /* Center horizontally */
   background: #fff;
   box-shadow: 0 2px 5px rgba(0,0,0,0.1);
   /* Removed border-radius and overflow for simplicity */
 }

 .app-header {
    background-color: var(--header-bg);
    padding: 1em;
    border-bottom: 1px solid var(--border-color);
    text-align: center;
 }
  .app-header h1 {
     margin: 0 0 0.5em 0;
     font-weight: 600;
  }

 .main-content {
   display: flex;
   flex-direction: row;
   padding: 1em;
   gap: 1.5em; /* Increased gap */
   flex-grow: 1; /* Allow main content to grow */
 }

 .game-area {
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1em; /* Gap between grid and number pad */
 }

 .start-prompt {
     margin-top: 50px;
     padding: 20px;
     text-align: center;
     color: #555;
     font-size: 1.1em;
 }

 .loading-overlay {
     position: fixed;
     top: 0;
     left: 0;
     width: 100%;
     height: 100%;
     background-color: rgba(255, 255, 255, 0.75);
     display: flex;
     justify-content: center;
     align-items: center;
     font-size: 1.5em;
     color: #333;
     z-index: 1000;
 }

 .sidebar {
    flex-basis: 280px; /* Slightly wider sidebar */
    flex-shrink: 0;
    background-color: var(--sidebar-bg);
    padding: 1em;
    border-left: 1px solid var(--border-color);
 }

 @media (max-width: 800px) { /* Adjusted breakpoint */
     .main-content {
         flex-direction: column;
     }
     .sidebar {
         flex-basis: auto;
          width: 100%;
         border-left: none;
         border-top: 1px solid var(--border-color);
          padding: 0.5em;
     }
      .game-area {
         width: 100%;
     }
 }

 /* Basic Button Styles */
 button {
    padding: 8px 15px;
    border: 1px solid var(--border-color);
    border-radius: 4px;
    background-color: var(--button-bg);
    cursor: pointer;
    font-size: 0.95rem;
    transition: background-color 0.2s ease;
 }

  button:hover:not(:disabled) {
    background-color: var(--button-hover-bg);
 }

  button:disabled {
    background-color: var(--button-disabled-bg);
    color: var(--button-disabled-text);
    cursor: not-allowed;
    opacity: 0.7;
 }

  /* Add other base styles as needed */
</style>