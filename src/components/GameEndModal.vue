<!-- ./components/GameEndModal.vue -->
<script setup lang="ts"> // Added lang="ts"
import { ref, onMounted } from 'vue';

// Define Props interface
interface Props {
  score: number | null; // Score might be null if fetch failed
  difficulty: string;
  time: string;
}
const props = defineProps<Props>();

// Define Emits type literal
const emit = defineEmits<{
  (event: 'submitScore', username: string): void;
  (event: 'close'): void;
}>();

// Local state with types
const username = ref<string>('');
const usernameInput = ref<HTMLInputElement | null>(null); // Ref for autofocus

// Event handlers with types
const handleSubmit = (): void => {
    const trimmedUsername = username.value.trim();
    if (trimmedUsername) {
      emit('submitScore', trimmedUsername);
    } else {
        alert("Please enter a username.");
        usernameInput.value?.focus(); // Optional chaining for safety
    }
};

const closeModal = (): void => {
     emit('close');
};

// Autofocus the input when the modal appears
onMounted(() => {
  // Timeout helps ensure element is rendered after modal transition/display
  setTimeout(() => {
      usernameInput.value?.focus();
  }, 50); // Small delay
});

</script>

<template>
  <div class="modal-overlay" @click.self="closeModal">
    <div class="modal-content">
      <h2>🎉 Game Completed! 🎉</h2>
      <p>Difficulty: <strong>{{ difficulty }}</strong></p>
      <p>Time: <strong>{{ time }}</strong></p>
      <p>Final Score: <strong>{{ score?.toLocaleString() ?? 'N/A' }}</strong></p> <!-- Handle null score -->

      <!-- Use form for better accessibility and enter key submission -->
      <form @submit.prevent="handleSubmit">
          <div class="form-group">
            <label for="username">Enter your name for the leaderboard:</label>
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
            <button type="button" @click="closeModal" class="close-btn">Close</button> <!-- Changed to type="button" -->
          </div>
      </form>
    </div>
  </div>
</template>

<style scoped>
/* Styles remain the same as provided in the dump */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 30px 40px;
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 5px 15px rgba(0,0,0,0.2);
   width: 90%;
   max-width: 400px;
}

.modal-content h2 {
  margin-top: 0;
  color: #333;
}

 .modal-content p {
    margin: 10px 0;
    font-size: 1.1em;
    color: #555;
 }
  .modal-content p strong {
    color: #0056b3;
 }

.form-group {
    margin: 20px 0;
    text-align: left;
}
.form-group label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
    font-size: 0.9em;
}
 .form-group input {
    width: 100%;
    padding: 10px;
    border: 1px solid var(--border-color);
    border-radius: 4px;
    box-sizing: border-box; /* Include padding in width */
    background-color: white;
    color: black;
 }

.modal-actions {
  margin-top: 25px;
  display: flex;
  justify-content: space-around; /* Or center, space-between */
   gap: 10px;
}

.modal-actions button {
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1em;
    min-width: 100px;
}

 .submit-btn {
    background-color: #28a745; /* Green */
    color: white;
 }
 .submit-btn:hover {
    background-color: #218838;
 }

 .close-btn {
    background-color: #6c757d; /* Gray */
    color: white;
 }
 .close-btn:hover {
    background-color: #5a6268;
 }
</style>