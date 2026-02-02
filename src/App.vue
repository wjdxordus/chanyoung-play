<script setup>
import SlidingPuzzle from './components/SlidingPuzzle.vue'
</script>

<template>
  <div class="app">
    <div class="bubbles">
      <span class="bubble"></span>
      <span class="bubble"></span>
      <span class="bubble"></span>
      <span class="bubble"></span>
      <span class="bubble"></span>
    </div>
    <SlidingPuzzle />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, sans-serif;
  background: linear-gradient(180deg, #fce4ec 0%, #fff8e1 50%, #e3f2fd 100%);
  min-height: 100vh;
  overflow-x: hidden;
}

.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 16px;
  min-height: 100vh;
  position: relative;
}

.bubbles {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  overflow: hidden;
  z-index: 0;
}

.bubble {
  position: absolute;
  border-radius: 50%;
  opacity: 0.6;
  animation: float 8s ease-in-out infinite;
}

.bubble:nth-child(1) {
  width: 60px;
  height: 60px;
  background: radial-gradient(circle, #ffcdd2 0%, #f8bbd9 100%);
  top: 10%;
  left: 10%;
  animation-delay: 0s;
}

.bubble:nth-child(2) {
  width: 40px;
  height: 40px;
  background: radial-gradient(circle, #b3e5fc 0%, #81d4fa 100%);
  top: 20%;
  right: 15%;
  animation-delay: 1s;
}

.bubble:nth-child(3) {
  width: 50px;
  height: 50px;
  background: radial-gradient(circle, #fff9c4 0%, #fff176 100%);
  bottom: 25%;
  left: 8%;
  animation-delay: 2s;
}

.bubble:nth-child(4) {
  width: 35px;
  height: 35px;
  background: radial-gradient(circle, #c8e6c9 0%, #a5d6a7 100%);
  bottom: 15%;
  right: 12%;
  animation-delay: 3s;
}

.bubble:nth-child(5) {
  width: 45px;
  height: 45px;
  background: radial-gradient(circle, #e1bee7 0%, #ce93d8 100%);
  top: 50%;
  right: 5%;
  animation-delay: 4s;
}

@keyframes float {
  0%, 100% {
    transform: translateY(0) scale(1);
  }
  50% {
    transform: translateY(-20px) scale(1.1);
  }
}
</style>
