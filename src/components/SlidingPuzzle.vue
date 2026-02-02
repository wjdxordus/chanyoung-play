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
  background: white;
  padding: 20px;
  border-radius: 16px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
}

.timer {
  margin-bottom: 16px;
  padding: 8px 20px;
  background: #eee;
  border-radius: 20px;
  font-size: 1.5rem;
  font-weight: bold;
  font-family: monospace;
}

.timer.running {
  background: #667eea;
  color: white;
}

.timer.completed {
  background: #4caf50;
  color: white;
}

.puzzle-board {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 2px;
  background: #333;
  padding: 2px;
  border-radius: 8px;
  width: 320px;
  height: 320px;
}

.tile {
  background: #ddd;
  border-radius: 4px;
}

.tile.clickable {
  cursor: pointer;
  box-shadow: 0 0 8px rgba(102, 126, 234, 0.6);
}

.tile.clickable:hover {
  transform: scale(0.96);
}

.tile.empty {
  background: #222;
}

.complete-message {
  margin-top: 16px;
  padding: 10px 20px;
  background: #4caf50;
  color: white;
  border-radius: 8px;
  font-weight: bold;
}

.btn {
  margin-top: 16px;
  padding: 10px 24px;
  font-size: 1rem;
  font-weight: bold;
  color: white;
  background: #667eea;
  border: none;
  border-radius: 20px;
  cursor: pointer;
}

.btn:hover {
  background: #5a6fd6;
}

.preview {
  margin-top: 20px;
}

.preview img {
  width: 100px;
  height: 100px;
  object-fit: cover;
  border-radius: 8px;
  border: 2px solid #eee;
}
</style>
