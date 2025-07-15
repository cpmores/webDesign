<template>
  <div class="login-portal">
    <div class="logo-container" ref="logoContainer">
      <ArcText />
      <LogoCore class="logo-core" ref="logoCore" />
    </div>
    <div class="btn-container" ref="btnContainer">
      <button class="portal-btn" @click="showLoginModal = true">Login</button>
      <button class="portal-btn" @click="showRegisterModal = true">Register</button>
    </div>
    <LoginModal :show="showLoginModal" @close="showLoginModal = false" />
    <RegisterModal :show="showRegisterModal" @close="showRegisterModal = false" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { gsap, Power3 } from 'gsap';
import ArcText from '../components/ArcText.vue';
import LogoCore from '../components/LogoCore.vue';
import LoginModal from '../components/LoginModal.vue';
import RegisterModal from '../components/RegisterModal.vue';

const showLoginModal = ref(false);
const showRegisterModal = ref(false);
const logoContainer = ref(null);
const logoCore = ref(null);
const btnContainer = ref(null);

onMounted(() => {
  gsap.set(logoContainer.value, { scale: 1.5 });
  gsap.set(btnContainer.value, { opacity: 0, y: 50 });

  const timeline = gsap.timeline();
  timeline
    .add(() => logoCore.value.animate())
    .to(logoContainer.value, {
      y: -50,
      duration: 0.8,
      ease: Power3.out,
      onComplete: () => {
        gsap.to(btnContainer.value, {
          opacity: 1,
          y: 0,
          duration: 0.6,
          ease: Power3.out
        });
      }
    });
});
</script>

<style scoped>
.login-portal {
  width: 100vw;
  height: 100vh;
  background-color: #fdf1e9;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  overflow: visible;
  position: relative;
}

.logo-container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 340px;
  height: 34vh;
  margin-bottom: 0px;
}

.logo-core {
  position: absolute;
  top: 13vh;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1; /* Place behind ArcText to avoid overlap */
  width: 100px; /* Adjust based on LogoCore's size */
  height: 100px; /* Adjust based on LogoCore's size */
}

.btn-container {
  margin-top: 0px;
}

.portal-btn {
  background-color: #663311;
  color: #fff;
  border: none;
  padding: 10px 28px;
  margin: 0 18px;
  border-radius: 24px;
  font-size: 1.2rem;
  font-family: 'ReadexPro', serif;
  width: 130px;
  cursor: pointer;
  box-shadow: 0 2px 8px #0001;
  transition: background 0.2s, transform 0.2s;
}

.portal-btn:hover {
  background: #181818;
  transform: translateY(-2px) scale(1.04);
}
</style>