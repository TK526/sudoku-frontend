<script setup lang="ts">
import type { LeaderboardData, LeaderboardEntry } from '../types';

interface Props {
  leaderboardData: LeaderboardData | null; // Can be null initially
  isLoading: boolean;
}
defineProps<Props>();

const difficultyOrder: Array<keyof LeaderboardData> = ['beginner', 'intermediate', 'hard', 'expert'];

// Function to format score
const formatScore = (score: number): string => score?.toLocaleString() ?? 'N/A';

// Helper to capitalize difficulty names
const capitalize = (s: string): string => s.charAt(0).toUpperCase() + s.slice(1);

</script>

<template>
  <div class="leaderboard">
    <h3>🏆 Leaderboard</h3>
    <div v-if="isLoading" class="loading-message">Loading...</div>
    <div v-else-if="!leaderboardData || difficultyOrder.every(d => !leaderboardData[d]?.length)" class="no-data">
       No records yet. Play a game!
    </div>
    <div v-else>
       <!-- Iterate over difficulties in defined order -->
       <div v-for="difficulty in difficultyOrder" :key="difficulty" class="difficulty-section">
         <h4>{{ capitalize(difficulty) }}</h4>
         <ul v-if="leaderboardData[difficulty]?.length > 0">
           <li v-for="(record, index) in leaderboardData[difficulty]" :key="record.id">
             <span class="rank">{{ index + 1 }}.</span>
             <span class="username">{{ record.username }}</span>
             <span class="score">{{ formatScore(record.score) }}</span>
           </li>
         </ul>
         <p v-else class="no-records">No records for {{ difficulty }}.</p>
       </div>
    </div>
  </div>
</template>

<style scoped>
.leaderboard {
  text-align: left;
  padding: 10px;
  background-color: var(--sidebar-bg); /* Ensure background */
  border-radius: 4px; /* Optional rounded corners */
}

.leaderboard h3 {
  text-align: center;
  margin-top: 0;
  margin-bottom: 15px;
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 8px; /* Increased padding */
  font-weight: 600;
  color: #333;
}

.difficulty-section {
    margin-bottom: 18px; /* Increased spacing */
}

.difficulty-section:last-child {
    margin-bottom: 0;
}

 .difficulty-section h4 {
    margin-bottom: 8px; /* Increased spacing */
    color: #444;
    font-size: 1em;
    font-weight: 600;
    border-bottom: 1px solid #eee; /* Subtle separator */
    padding-bottom: 3px;
 }

ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

li {
  display: grid; /* Use grid for alignment */
  grid-template-columns: auto 1fr auto; /* Rank, Name (flexible), Score */
  gap: 8px; /* Spacing between columns */
  padding: 5px 0; /* Increased padding */
  font-size: 0.9em;
  border-bottom: 1px dashed #e0e0e0; /* Slightly darker dash */
  align-items: center;
}

li:last-child {
    border-bottom: none;
}

.rank {
    color: #666;
    font-weight: 500;
    min-width: 20px; /* Ensure rank aligns */
    text-align: right;
}

.username {
    font-weight: 500;
    color: #333;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis; /* Prevent long names breaking layout */
}

.score {
    color: var(--color-success); /* Use variable */
    font-weight: 600;
    text-align: right;
}

 .loading-message, .no-data, .no-records {
    text-align: center;
    color: #777;
    padding: 15px 0; /* More padding */
    font-style: italic;
 }
 .no-records {
     font-size: 0.85em;
     padding: 5px 0;
 }
</style>