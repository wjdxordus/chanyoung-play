<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'

const GRID_SIZE = 4
const TOTAL_TILES = GRID_SIZE * GRID_SIZE
const EMPTY_TILE = TOTAL_TILES - 1

const baseUrl = import.meta.env.BASE_URL || '/'
const imagePath = `${baseUrl}IMG_5861.JPG`

const tiles = ref([])
const elapsedTime = ref(0)
const isTimerRunning = ref(false)
const hasStarted = ref(false)
let timerInterval = null

const isComplete = computed(() => {
  return tiles.value.every((tile, index) => tile === index)
})

const formattedTime = computed(() => {
  const minutes = Math.floor(elapsedTime.value / 60)
  const seconds = elapsedTime.value % 60
  return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
})

const startTimer = () => {
  if (isTimerRunning.value) return
  isTimerRunning.value = true
  hasStarted.value = true
  timerInterval = setInterval(() => {
    elapsedTime.value++
  }, 1000)
}

const stopTimer = () => {
  if (timerInterval) {
    clearInterval(timerInterval)
    timerInterval = null
  }
  isTimerRunning.value = false
}

const resetTimer = () => {
  stopTimer()
  elapsedTime.value = 0
  hasStarted.value = false
}

watch(isComplete, (completed) => {
  if (completed && isTimerRunning.value) {
    stopTimer()
  }
})

const getTileStyle = (tileValue) => {
  if (tileValue === EMPTY_TILE) {
    return { visibility: 'hidden' }
  }

  const row = Math.floor(tileValue / GRID_SIZE)
  const col = tileValue % GRID_SIZE

  return {
    backgroundImage: `url(${imagePath})`,
    backgroundSize: `${GRID_SIZE * 100}% ${GRID_SIZE * 100}%`,
    backgroundPosition: `${(col / (GRID_SIZE - 1)) * 100}% ${(row / (GRID_SIZE - 1)) * 100}%`
  }
}

const findEmptyIndex = () => {
  return tiles.value.indexOf(EMPTY_TILE)
}

const isAdjacent = (index) => {
  const emptyIndex = findEmptyIndex()
  const emptyRow = Math.floor(emptyIndex / GRID_SIZE)
  const emptyCol = emptyIndex % GRID_SIZE
  const tileRow = Math.floor(index / GRID_SIZE)
  const tileCol = index % GRID_SIZE

  const rowDiff = Math.abs(emptyRow - tileRow)
  const colDiff = Math.abs(emptyCol - tileCol)

  return (rowDiff === 1 && colDiff === 0) || (rowDiff === 0 && colDiff === 1)
}

const handleTileClick = (index) => {
  if (isComplete.value) return

  if (isAdjacent(index)) {
    if (!hasStarted.value) {
      startTimer()
    }

    const emptyIndex = findEmptyIndex()
    const newTiles = [...tiles.value]
    ;[newTiles[index], newTiles[emptyIndex]] = [newTiles[emptyIndex], newTiles[index]]
    tiles.value = newTiles
  }
}

const isSolvable = (arr) => {
  let inversions = 0
  const flatArr = arr.filter(x => x !== EMPTY_TILE)

  for (let i = 0; i < flatArr.length; i++) {
    for (let j = i + 1; j < flatArr.length; j++) {
      if (flatArr[i] > flatArr[j]) {
        inversions++
      }
    }
  }

  const emptyRow = Math.floor(arr.indexOf(EMPTY_TILE) / GRID_SIZE)
  const emptyRowFromBottom = GRID_SIZE - emptyRow

  if (emptyRowFromBottom % 2 === 1) {
    return inversions % 2 === 0
  } else {
    return inversions % 2 === 1
  }
}

const shuffleArray = (arr) => {
  const shuffled = [...arr]
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
  }
  return shuffled
}

const initGame = () => {
  let shuffled
  const original = Array.from({ length: TOTAL_TILES }, (_, i) => i)

  do {
    shuffled = shuffleArray(original)
  } while (!isSolvable(shuffled) || shuffled.every((val, idx) => val === idx))

  tiles.value = shuffled
  resetTimer()
}

onMounted(() => {
  initGame()
})

onUnmounted(() => {
  stopTimer()
})
</script>

<template>
  <div class="puzzle-container">
    <div class="timer" :class="{ running: isTimerRunning, completed: isComplete }">
      {{ formattedTime }}
    </div>

    <div class="puzzle-board">
      <div
        v-for="(tile, index) in tiles"
        :key="index"
        class="tile"
        :class="{
          empty: tile === EMPTY_TILE,
          clickable: isAdjacent(index) && !isComplete
        }"
        :style="getTileStyle(tile)"
        @click="handleTileClick(index)"
      />
    </div>

    <div v-if="isComplete" class="complete-message">
      {{ formattedTime }} 완성!
    </div>

    <button @click="initGame" class="btn">새 게임</button>

    <div class="preview">
      <img :src="imagePath" alt="원본" />
    </div>
  </div>
</template>

<style scoped>
.puzzle-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: rgba(255, 255, 255, 0.9);
  padding: 20px;
  border-radius: 24px;
  box-shadow: 0 8px 32px rgba(255, 183, 197, 0.3);
  border: 3px solid #ffcdd2;
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 360px;
}

.timer {
  margin-bottom: 16px;
  padding: 10px 24px;
  background: linear-gradient(135deg, #fff3e0 0%, #ffe0b2 100%);
  border-radius: 20px;
  font-size: 1.4rem;
  font-weight: bold;
  font-family: monospace;
  color: #ff8a65;
  border: 2px solid #ffcc80;
}

.timer.running {
  background: linear-gradient(135deg, #ffcdd2 0%, #f8bbd9 100%);
  color: white;
  border-color: #f48fb1;
}

.timer.completed {
  background: linear-gradient(135deg, #c8e6c9 0%, #a5d6a7 100%);
  color: white;
  border-color: #81c784;
}

.puzzle-board {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 2px;
  background: linear-gradient(135deg, #f8bbd9 0%, #ffcdd2 100%);
  padding: 4px;
  border-radius: 16px;
  width: 100%;
  max-width: 320px;
  aspect-ratio: 1;
}

.tile {
  background: #fff;
  border-radius: 8px;
  transition: transform 0.15s ease;
}

.tile.clickable {
  cursor: pointer;
  box-shadow: 0 0 12px rgba(244, 143, 177, 0.6);
}

.tile.clickable:hover {
  transform: scale(0.95);
}

.tile.empty {
  background: rgba(255, 255, 255, 0.3);
}

.complete-message {
  margin-top: 16px;
  padding: 12px 24px;
  background: linear-gradient(135deg, #c8e6c9 0%, #a5d6a7 100%);
  color: white;
  border-radius: 16px;
  font-weight: bold;
  font-size: 1.1rem;
}

.btn {
  margin-top: 16px;
  padding: 12px 28px;
  font-size: 1rem;
  font-weight: bold;
  color: white;
  background: linear-gradient(135deg, #f8bbd9 0%, #f48fb1 100%);
  border: none;
  border-radius: 24px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.btn:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 16px rgba(244, 143, 177, 0.4);
}

.btn:active {
  transform: scale(0.98);
}

.preview {
  margin-top: 20px;
}

.preview img {
  width: 160px;
  height: 160px;
  object-fit: cover;
  border-radius: 16px;
  border: 3px solid #ffcdd2;
}

@media (max-width: 400px) {
  .puzzle-container {
    padding: 16px;
    max-width: 100%;
  }

  .puzzle-board {
    max-width: 280px;
  }

  .timer {
    font-size: 1.2rem;
    padding: 8px 20px;
  }

  .preview img {
    width: 120px;
    height: 120px;
  }
}
</style>
