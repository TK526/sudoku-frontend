<script setup>
import { ref } from 'vue';

const props = defineProps({
    score: Number,
    difficulty: String,
    time: String,
});

const emit = defineEmits(['submit-score', 'close']);

const username = ref('');

const handleSubmit = () => {
    if (username.value.trim()) {
        emit('submit-score', username.value.trim());
    } else {
        alert("Please enter a username.");
    }
};

const closeModal = () => {
     emit('close');
};
</script>

<template>
  <div class="modal-overlay" @click.self="closeModal"> <!-- Close on overlay click -->
    <div class="modal-content">
      <h2>🎉 Game Completed! 🎉</h2>
      <p>Difficulty: <strong>{{ difficulty }}</strong></p>
      <p>Time: <strong>{{ time }}</strong></p>
      <p>Final Score: <strong>{{ score?.toLocaleString() ?? 'Calculating...' }}</strong></p>

      <div class="form-group">
        <label for="username">Enter your name for the leaderboard:</label>
        <input type="text" id="username" v-model="username" placeholder="Your Name" maxlength="20" required>
      </div>

      <div class="modal-actions">
        <button @click="handleSubmit" class="submit-btn">Submit Score</button>
        <button @click="closeModal" class="close-btn">Close</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
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