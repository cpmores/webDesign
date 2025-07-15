<template>
  <transition name="modal">
    <div class="modal-mask" v-if="show">
      <div class="modal-container">
        <div class="modal-header">
          <h3>Register</h3>
        </div>
        <div class="modal-body">
          <p><input type="text" placeholder="Username" /></p>
          <p><input type="email" placeholder="Email" /></p>
          <p><input type="password" placeholder="Password" /></p>
          <p><input type="password" placeholder="Confirmed Password" /></p>
        </div>
        <div class="modal-footer">
          <button class="modal-btn" @click="emit('close')">Cancel</button>
          <button class="modal-btn" @click="handleRegister">Register</button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';

defineProps({
  show: Boolean
});

const emit = defineEmits(['close']);

const handleRegister = () => {
  console.log('执行注册操作');
  emit('close');
};
</script>

<style scoped>
.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-container {
  width: 400px;
  max-width: 90%;
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  overflow: hidden;
  font-family: 'ReadexPro', Arial, sans-serif;
  justify-content: center;
  align-items: center;
}

.modal-header {
  padding: 20px;
  background-color: #fff;
  border-bottom: 1px solid #eaeaea;
}

.modal-header h3 {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
  color: #333;
}

.modal-body {
  padding: 20px;
}

.modal-body p {
  margin-bottom: 15px;
  margin-right: 15px;
}

.modal-body input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 1rem;
}

.modal-footer {
  padding: 15px 20px;
  background-color: #f9f9f9;
  border-top: 1px solid #eaeaea;
  display: flex;
  justify-content: flex-end;
}

.modal-btn {
  margin-left: 10px;
  padding: 8px 16px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  font-family: 'ReadexPro';
  transition: background-color 0.2s;
}

.modal-btn:hover {
  opacity: 0.9;
}

.modal-btn:first-child {
  background-color: #f0f0f0;
  color: #333;
}

.modal-btn:last-child {
  background-color: #000;
  color: #fff;
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}
</style>