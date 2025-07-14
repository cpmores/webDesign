<template>
  <div class="login-portal">
    <!-- LOGO动画整体放大 -->
    <div class="logo-container" ref="logoContainer">
      <!-- 环绕文字（SVG弧形） -->
      <svg class="logo-arc-text" width="340" height="220">
        <defs>
          <path id="arc-top" d="M40, 300 A130,220 0 0,1 300,300" fill="none" />
          <path id="arc-bottom" d="M60,180 A110,110 0 0,0 280,180" fill="none" />
        </defs>
        <text fill="#470024" font-size="32" font-family="'ReadexPro'" font-weight="bold">
          <textPath xlink:href="#arc-top" startOffset="50%" text-anchor="middle">WISDOMWEB</textPath>
        </text>
        <text fill="#470024" font-size="18" font-family="'ReadexPro'">
          <textPath xlink:href="#arc-bottom" startOffset="50%" text-anchor="middle">CONNECTING KNOWLEDGE SEAMLESSLY</textPath>
        </text>
      </svg>
      <!-- logo主体 -->
      <div class="logo-core">
        <div class="database" ref="databaseTop" :style="{ backgroundColor: databaseColor }"></div>
        <div class="database" ref="databaseMiddle" :style="{ backgroundColor: databaseColor_white }"></div>
        <div class="database" ref="databaseBottom" :style="{ backgroundColor: databaseColor }"></div>
        <div class="deco-star" ref="starLeft"></div>
        <div class="deco-star" ref="starRight"></div>
      </div>
    </div>
    <!-- 登录和注册按钮容器，初始隐藏，LOGO 上移后显示 -->
    <div class="btn-container" ref="btnContainer">
      <button class="portal-btn" ref="loginBtn" @click="showLoginModal =!showLoginModal">Login</button>
      <button class="portal-btn" ref="registerBtn" @click="showRegisterModal =!showRegisterModal">Register</button>
    </div>
<!-- 登录弹窗 -->
<transition name="modal">
  <div class="modal-mask" v-if="showLoginModal">
    <div class="modal-container">
      <div class="modal-header">
        <h3>Login</h3>
      </div>
      <div class="modal-body">
        <p><input type="text" placeholder="Username" /></p>
        <p><input type="password" placeholder="Password" /></p>
      </div>
      <div class="modal-footer">
        <button class="modal-btn" @click="showLoginModal = false">Cancel</button>
        <button class="modal-btn" @click="handleLogin">Login</button>
      </div>
    </div>
  </div>
</transition>

<!-- 注册弹窗 -->
<transition name="modal">
  <div class="modal-mask" v-if="showRegisterModal">
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
        <button class="modal-btn" @click="showRegisterModal = false">Cancel</button>
        <button class="modal-btn" @click="handleRegister">Register</button>
      </div>
    </div>
  </div>
</transition>
  </div>
</template>

<script setup>
import { gsap, Power3 } from 'gsap';
import { ref, onMounted } from 'vue';

const showLoginModal = ref(false);
const showRegisterModal = ref(false);
const databaseColor = ref('#000'); // 数据库图形颜色，匹配 logo 黑色
const databaseColor_white = ref('#470024'); // 数据库图形颜色，匹配 logo 黑色
const logoContainer = ref(null);
const databaseTop = ref(null);
const databaseMiddle = ref(null);
const databaseBottom = ref(null);
const starLeft = ref(null);
const starRight = ref(null);
const btnContainer = ref(null);

onMounted(() => {
  initAnimation();
});

function initAnimation() {
  // 初始隐藏相关元素
  gsap.set([databaseTop.value, databaseBottom.value], { x: -100, opacity: 0 });
  gsap.set(databaseMiddle.value, { x: 100, opacity: 0 });
  gsap.set([starLeft.value, starRight.value], { opacity: 0, scale: 0 });
  gsap.set(btnContainer.value, { opacity: 0, y: 50 });
  gsap.set(logoContainer.value, { scale: 1.5 }); // 整体放大

  // 数据库图形动画
  const databaseTopAnim = gsap.to(databaseTop.value, {
    x: 0,
    opacity: 1,
    duration: 0.8,
    ease: Power3.out
  });
  const databaseBottomAnim = gsap.to(databaseBottom.value, {
    x: 0,
    opacity: 1,
    duration: 0.8,
    ease: Power3.out
  });
  const databaseMiddleAnim = gsap.to(databaseMiddle.value, {
    x: 0,
    opacity: 1,
    duration: 0.8,
    ease: Power3.out,
    delay: 0.3
  });

  // 星星装饰动画
  const starLeftAnim = gsap.to(starLeft.value, {
    opacity: 1,
    scale: 1,
    duration: 0.6,
    ease: Power3.out,
    delay: 0.5
  });
  const starRightAnim = gsap.to(starRight.value, {
    opacity: 1,
    scale: 1,
    duration: 0.6,
    ease: Power3.out,
    delay: 0.5
  });

  // 图形动画完成后执行整体上移动画
  const timeline = gsap.timeline();
  timeline
   .add([databaseTopAnim, databaseBottomAnim, databaseMiddleAnim, starLeftAnim, starRightAnim])
   .to(logoContainer.value, {
      y: -50,
      duration: 0.8,
      ease: Power3.out,
      onComplete: () => {
        // LOGO 上移后显示按钮
        gsap.to(btnContainer.value, {
          opacity: 1,
          y: 0,
          duration: 0.6,
          ease: Power3.out
        });
      }
    });
}

function handleLogin() {
  // 可在此添加登录逻辑，比如调用接口等
  console.log('执行登录操作');
  showLoginModal.value = false;
}

function handleRegister() {
  // 可在此添加注册逻辑，比如调用接口等
  console.log('执行注册操作');
  showRegisterModal.value = false;
}
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
  /* scale 由 gsap 控制 */
}
.logo-arc-text {
  position: absolute;
  left: 0;
  top: 0;
  pointer-events: none;
  width: 100%;
  height: 100%;
  z-index: 2;
}
.logo-core {
  position: absolute;
  left: 50%;
  top: 57%;
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
.database:nth-child(1) { margin-bottom: 5px; }
.database:nth-child(2) { margin-bottom: 5px; }
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
/* 弹窗样式同前略 */
.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7); /* 更深的遮罩 */
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-container {
  width: 400px; /* 更宽 */
  max-width: 90%; /* 响应式 */
  background-color: #fff;
  border-radius: 12px; /* 更大的圆角 */
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2); /* 更强的阴影 */
  overflow: hidden;
  font-family: 'ReadexPro', Arial, sans-serif; /* 更现代的字体 */
  justify-content: center;
  align-items: center;
}

.modal-header {
  padding: 20px;
  background-color: #fff; /* 白色背景 */
  border-bottom: 1px solid #eaeaea; /* 底部边框 */
}

.modal-header h3 {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
  color: #333; /* 深灰色文字 */
}

.modal-body {
  padding: 20px;
}

.modal-body p {
  margin-bottom: 15px;
  margin-right: 15px
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
  background-color: #f9f9f9; /* 浅灰色背景 */
  border-top: 1px solid #eaeaea; /* 顶部边框 */
  display: flex;
  justify-content: flex-end; /* 按钮右对齐 */
}

.modal-btn {
  margin-left: 10px; /* 按钮间距 */
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

/* 取消按钮样式 */
.modal-btn:first-child {
  background-color: #f0f0f0;
  color: #333;
}

/* 确认按钮样式 */
.modal-btn:last-child {
  background-color: #000; /* 黑色按钮 */
  color: #fff;
}

/* 过渡动画 */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}
</style>