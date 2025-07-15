<script setup>
import WebGLWaveBg from './components/WebGLWaveBg.vue';
import LoginPortal from './views/LoginPortal.vue'
import MainView from './views/MainView.vue';
</script>

<template>
    <WebGLWaveBg/>
    <MainView/>
  <!-- <LoginPortal /> -->
</template>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>