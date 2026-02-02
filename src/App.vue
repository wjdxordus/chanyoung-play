<script setup>
import SlidingPuzzle from './components/SlidingPuzzle.vue'
</script>

<template>
  <div class="app">
    <div class="decorations">
      <span class="star star1">&#9733;</span>
      <span class="star star2">&#9733;</span>
      <span class="star star3">&#9733;</span>
      <span class="cloud cloud1">&#9729;</span>
      <span class="cloud cloud2">&#9729;</span>
      <span class="heart heart1">&#9829;</span>
      <span class="heart heart2">&#9829;</span>
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
  font-family: 'Comic Sans MS', 'Chalkboard', cursive, sans-serif;
  background: linear-gradient(180deg, #ffe6f0 0%, #fff0d4 50%, #e6f7ff 100%);
  min-height: 100vh;
  overflow-x: hidden;
}

.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
  min-height: 100vh;
  position: relative;
}

.decorations {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 0;
}

.star {
  position: absolute;
  color: #ffd700;
  opacity: 0.6;
  animation: twinkle 2s ease-in-out infinite;
}

.star1 { top: 10%; left: 10%; font-size: 2rem; animation-delay: 0s; }
.star2 { top: 20%; right: 15%; font-size: 1.5rem; animation-delay: 0.5s; }
.star3 { bottom: 30%; left: 8%; font-size: 1.8rem; animation-delay: 1s; }

.cloud {
  position: absolute;
  color: white;
  opacity: 0.8;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  animation: float 6s ease-in-out infinite;
}

.cloud1 { top: 8%; right: 20%; font-size: 4rem; animation-delay: 0s; }
.cloud2 { top: 15%; left: 5%; font-size: 3rem; animation-delay: 2s; }

.heart {
  position: absolute;
  color: #ffb6c1;
  opacity: 0.5;
  animation: pulse 2s ease-in-out infinite;
}

.heart1 { bottom: 20%; right: 10%; font-size: 2rem; animation-delay: 0s; }
.heart2 { top: 30%; right: 5%; font-size: 1.5rem; animation-delay: 1s; }

@keyframes twinkle {
  0%, 100% { opacity: 0.6; transform: scale(1); }
  50% { opacity: 1; transform: scale(1.2); }
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

@keyframes pulse {
  0%, 100% { opacity: 0.5; transform: scale(1); }
  50% { opacity: 0.8; transform: scale(1.1); }
}
</style>
