<script setup lang="ts">
import { ref, onMounted } from 'vue';

interface Props {
  score: number | null; // Score might not be ready immediately
  difficulty: string;
  time: string;
}
const props = defineProps<Props>();

const emit = defineEmits<{
  (event: 'submitScore', username: string): void;
  (event: 'close'): void;
}>();

const username = ref<string>('');
const usernameInput = ref<HTMLInputElement | null>(null); // Ref for autofocus

const handleSubmit = () => {
  const trimmedUsername = username.value.trim();
  if (trimmedUsername) {
    emit('submitScore', trimmedUsername);
  } else {
    alert("Please enter a username.");
    usernameInput.value?.focus(); // Focus input if empty
  }
};

const handleClose = () => {
  emit('close');
};

// Autofocus the input when the modal appears
onMounted(() => {
  usernameInput.value?.focus();
});

</script>

<template>
  <div class="modal-overlay" @click.self="handleClose"> <!-- Close on overlay click -->
    <div class="modal-content" role="dialog" aria-modal="true" aria-labelledby="modal-title">
      <h2 id="modal-title">🎉 Game Completed! 🎉</h2>
      <div class="results">
          <p>Difficulty: <strong>{{ difficulty }}</strong></p>
          <p>Time: <strong>{{ time }}</strong></p>
          <p>Final Score: <strong class="final-score">{{ score?.toLocaleString() ?? 'Calculating...' }}</strong></p>
      </div>

      <form @submit.prevent="handleSubmit" class="score-form">
        <div class="form-group">
          <label for="username">Enter name for Leaderboard:</label>
          <input
            ref="usernameInput"
            type="text"
            id="username"
            v-model="username"
            placeholder="Your Name"
            maxlength="20"
            required
          >
        </div>

        <div class="modal-actions">
          <button type="submit" class="submit-btn">Submit Score</button>
          <button type="button" @click="handleClose" class="close-btn">Close</button>
        </div>
      </form>
    </div>
  </div>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background-color: rgba(0, 0, 0, 0.65);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 1em; /* Add padding for smaller screens */
}

.modal-content {
  background-color: white;
  padding: 25px 35px; /* Adjusted padding */
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 5px 15px rgba(0,0,0,0.25);
  width: 100%;
  max-width: 420px; /* Slightly wider */
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
    from { opacity: 0; transform: scale(0.95); }
    to { opacity: 1; transform: scale(1); }
}


.modal-content h2 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #333;
  font-weight: 600;
}

.results p {
    margin: 8px 0; /* Reduced margin */
    font-size: 1.05em;
    color: #444;
 }
  .results strong {
    color: #0056b3;
    margin-left: 5px;
 }
 .final-score {
     font-size: 1.2em;
     color: var(--color-success);
 }

.score-form {
    margin-top: 20px;
}

.form-group {
    margin-bottom: 15px;
    text-align: left;
}
.form-group label {
    display: block;
    margin-bottom: 6px; /* Increased spacing */
    font-weight: 500;
    font-size: 0.9em;
    color: #555;
}
 .form-group input {
    width: 100%;
    padding: 10px 12px; /* Adjusted padding */
    border: 1px solid var(--border-color);
    border-radius: 4px;
    font-size: 1em;
 }

 .form-group input:focus {
     outline: 2px solid blue;
     outline-offset: 1px;
     border-color: blue; /* Optional */
 }

.modal-actions {
  margin-top: 20px;
  display: flex;
  justify-content: space-between; /* Pushes buttons apart */
   gap: 10px;
}

.modal-actions button {
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1em;
    font-weight: 500;
    flex-grow: 1; /* Make buttons share space */
    max-width: 180px; /* Prevent buttons getting too wide */
}

 .submit-btn {
    background-color: var(--color-success);
    color: white;
 }
 .submit-btn:hover {
    background-color: #218838;
 }

 .close-btn {
    background-color: #6c757d;
    color: white;
 }
 .close-btn:hover {
    background-color: #5a6268;
 }
</style>