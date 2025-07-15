<template>
  <div class="logo-core">
    <div class="database" ref="databaseTop" :style="{ backgroundColor: databaseColor }"></div>
    <div class="database" ref="databaseMiddle" :style="{ backgroundColor: databaseColorWhite }"></div>
    <div class="database" ref="databaseBottom" :style="{ backgroundColor: databaseColor }"></div>
    <div class="deco-star" ref="starLeft"></div>
    <div class="deco-star" ref="starRight"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { gsap, Power3 } from 'gsap';

const databaseColor = ref('#000');
const databaseColorWhite = ref('#470024');
const databaseTop = ref(null);
const databaseMiddle = ref(null);
const databaseBottom = ref(null);
const starLeft = ref(null);
const starRight = ref(null);

const animate = () => {
  gsap.set([databaseTop.value, databaseBottom.value], { x: -100, opacity: 0 });
  gsap.set(databaseMiddle.value, { x: 100, opacity: 0 });
  gsap.set([starLeft.value, starRight.value], { opacity: 0, scale: 0 });

  const timeline = gsap.timeline();
  timeline
    .to(databaseTop.value, { x: 0, opacity: 1, duration: 0.8, ease: Power3.out })
    .to(databaseBottom.value, { x: 0, opacity: 1, duration: 0.8, ease: Power3.out }, 0)
    .to(databaseMiddle.value, { x: 0, opacity: 1, duration: 0.8, ease: Power3.out, delay: 0.3 }, 0)
    .to([starLeft.value, starRight.value], { opacity: 1, scale: 1, duration: 0.6, ease: Power3.out, delay: 0.5 });
};

onMounted(animate);

defineExpose({ animate });
</script>

<style scoped>
.logo-core {
  /* position: absolute;
  left: 50%;
  top: 50%; */
  transform: translate(-50%, -50%);
  width: 120px;
  height: 160px;
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 3;
}

.database {
  width: 90px;
  height: 30px;
  border-radius: 10px;
  margin: 0;
  background: #181818;
}

.database:nth-child(1) {
  margin-bottom: 5px;
}

.database:nth-child(2) {
  margin-bottom: 5px;
}

.deco-star {
  width: 18px;
  height: 18px;
  background: url('data:image/svg+xml;utf8,<svg fill="black" viewBox="0 0 24 24"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21 12 17.77 5.82 21 7 14.14 2 9.27 8.91 8.26 12 2"/></svg>') no-repeat center;
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}

.deco-star:first-of-type {
  left: -38px;
}

.deco-star:last-of-type {
  right: -38px;
}
</style>