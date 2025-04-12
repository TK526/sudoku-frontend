<script setup>
defineProps({
  leaderboardData: { type: Object, required: true },
   isLoading: { type: Boolean, default: false }
});

 const difficultyOrder = ['beginner', 'intermediate', 'hard', 'expert'];

 // Function to format score or time if needed, e.g., add commas
 const formatScore = (score) => score?.toLocaleString() || 'N/A';

</script>

<template>
  <div class="leaderboard">
    <h3>🏆 Leaderboard</h3>
    <div v-if="isLoading" class="loading-message">Loading...</div>
    <div v-else-if="!leaderboardData || Object.keys(leaderboardData).length === 0" class="no-data">
       No leaderboard data available yet.
    </div>
    <div v-else>
       <div v-for="difficulty in difficultyOrder" :key="difficulty" class="difficulty-section">
         <h4>{{ difficulty.charAt(0).toUpperCase() + difficulty.slice(1) }}</h4>
         <ul v-if="leaderboardData[difficulty]?.length > 0">
           <li v-for="(record, index) in leaderboardData[difficulty]" :key="record.id || index">
             <span>{{ index + 1 }}. {{ record.username }}</span>
             <span>{{ formatScore(record.score) }}</span>
           </li>
         </ul>
         <p v-else class="no-records">No records for this difficulty.</p>
       </div>
    </div>
  </div>
</template>

<style scoped>
.leaderboard {
  text-align: left;
  padding: 10px; /* Add some padding inside the sidebar */
}

.leaderboard h3 {
  text-align: center;
  margin-top: 0;
  margin-bottom: 15px;
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 5px;
}

.difficulty-section {
    margin-bottom: 15px;
}

 .difficulty-section h4 {
    margin-bottom: 5px;
    color: #444;
    font-size: 1em;
 }

ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

li {
  display: flex;
  justify-content: space-between;
  padding: 4px 0;
  font-size: 0.9em;
  border-bottom: 1px dashed #eee;
}

li:last-child {
    border-bottom: none;
}

li span:first-child {
    font-weight: bold;
    color: #555;
}
 li span:last-child {
    color: #0056b3; /* Score color */
}

 .loading-message, .no-data, .no-records {
    text-align: center;
    color: #777;
    padding: 10px;
 }
</style>