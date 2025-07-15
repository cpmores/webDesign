<template>
  <canvas ref="canvas" class="webgl-wave-bg" />
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'

const canvas = ref()

// ---------- 着色器 ----------
const vertexShader = `
  uniform float uTime;
  varying vec2 vUv;
  void main() {
    vUv = uv;
    vec3 pos = position;
    float wave1 = sin(pos.x * 6.0 + uTime) * 0.10;
    float wave2 = sin(pos.y * 4.0 + uTime * 0.7) * 0.08;
    pos.z += wave1 + wave2;
    gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
  }
`

const fragmentShader = `
  varying vec2 vUv;
  void main() {
    vec3 top    = vec3(0.0, 0.0, 0.0);   // 亮蓝
    vec3 bottom = vec3(0.9, 0.9, 0.95);  // 亮灰
    gl_FragColor = vec4(mix(bottom, top, vUv.y), 1.0);
  }
`

// ---------- 初始化 ----------
onMounted(() => {
  const renderer = new THREE.WebGLRenderer({ canvas: canvas.value, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(window.devicePixelRatio)

  const scene = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 10)
  camera.position.z = 1.2

  const geo = new THREE.PlaneGeometry(2, 2, 64, 64)
  const mat = new THREE.ShaderMaterial({
    vertexShader,
    fragmentShader,
    uniforms: {
      uTime: { value: 0 }
    }
  })
  scene.add(new THREE.Mesh(geo, mat))

  // ---------- 关键：动画循环 ----------
  const clock = new THREE.Clock()
  const animate = () => {
    requestAnimationFrame(animate)
    mat.uniforms.uTime.value = clock.getElapsedTime() // ← 这就是让画面动起来的关键
    renderer.render(scene, camera)
  }
  animate()

  // ---------- 窗口自适应 ----------
  const onResize = () => {
    renderer.setSize(window.innerWidth, window.innerHeight)
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
  }
  window.addEventListener('resize', onResize)

  // ---------- 清理 ----------
  onUnmounted(() => {
    window.removeEventListener('resize', onResize)
  })
})
</script>

<style scoped>
.webgl-wave-bg {
  position: fixed;
  inset: 0;
  z-index: -1;
  pointer-events: none;
}
</style>