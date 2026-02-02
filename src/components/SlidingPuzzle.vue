<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'

const GRID_SIZE = 4
const TOTAL_TILES = GRID_SIZE * GRID_SIZE
const EMPTY_TILE = TOTAL_TILES - 1

// 이미지 경로 (public 폴더에서 참조)
const imagePath = '/IMG_5861.JPG'

// 타일 배열 (0-14: 이미지 조각, 15: 빈 칸)
const tiles = ref([])

// 타이머 관련
const elapsedTime = ref(0)
const isTimerRunning = ref(false)
const hasStarted = ref(false)
let timerInterval = null

// 게임 완료 여부
const isComplete = computed(() => {
  return tiles.value.every((tile, index) => tile === index)
})

// 시간 포맷팅 (분:초)
const formattedTime = computed(() => {
  const minutes = Math.floor(elapsedTime.value / 60)
  const seconds = elapsedTime.value % 60
  return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
})

// 타이머 시작
const startTimer = () => {
  if (isTimerRunning.value) return
  isTimerRunning.value = true
  hasStarted.value = true
  timerInterval = setInterval(() => {
    elapsedTime.value++
  }, 1000)
}

// 타이머 정지
const stopTimer = () => {
  if (timerInterval) {
    clearInterval(timerInterval)
    timerInterval = null
  }
  isTimerRunning.value = false
}

// 타이머 리셋
const resetTimer = () => {
  stopTimer()
  elapsedTime.value = 0
  hasStarted.value = false
}

// 퍼즐 완성 시 타이머 정지
watch(isComplete, (completed) => {
  if (completed && isTimerRunning.value) {
    stopTimer()
  }
})

// 타일 위치에서 배경 위치 계산
const getTileStyle = (tileValue, index) => {
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

// 빈 타일의 인덱스 찾기
const findEmptyIndex = () => {
  return tiles.value.indexOf(EMPTY_TILE)
}

// 클릭한 타일이 빈 타일과 인접한지 확인
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

// 타일 클릭 처리
const handleTileClick = (index) => {
  if (isComplete.value) return

  if (isAdjacent(index)) {
    // 첫 번째 유효한 클릭 시 타이머 시작
    if (!hasStarted.value) {
      startTimer()
    }

    const emptyIndex = findEmptyIndex()
    const newTiles = [...tiles.value]
    ;[newTiles[index], newTiles[emptyIndex]] = [newTiles[emptyIndex], newTiles[index]]
    tiles.value = newTiles
  }
}

// 퍼즐이 풀 수 있는지 확인 (역전 개수 계산)
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

  // 4x4 퍼즐: 빈 칸이 아래에서 홀수 행이면 역전 개수가 짝수여야 풀 수 있음
  if (emptyRowFromBottom % 2 === 1) {
    return inversions % 2 === 0
  } else {
    return inversions % 2 === 1
  }
}

// 배열 섞기
const shuffleArray = (arr) => {
  const shuffled = [...arr]
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
  }
  return shuffled
}

// 게임 초기화
const initGame = () => {
  let shuffled
  const original = Array.from({ length: TOTAL_TILES }, (_, i) => i)

  do {
    shuffled = shuffleArray(original)
  } while (!isSolvable(shuffled) || shuffled.every((val, idx) => val === idx))

  tiles.value = shuffled
  resetTimer()
}

// 게임 리셋
const resetGame = () => {
  initGame()
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
    <div class="stats">
      <span class="timer" :class="{ 'running': isTimerRunning, 'completed': isComplete }">
        {{ formattedTime }}
      </span>
    </div>

    <div class="puzzle-board">
      <div
        v-for="(tile, index) in tiles"
        :key="index"
        class="tile"
        :class="{
          'empty': tile === EMPTY_TILE,
          'clickable': isAdjacent(index) && !isComplete
        }"
        :style="getTileStyle(tile, index)"
        @click="handleTileClick(index)"
      >
      </div>
    </div>

    <div v-if="isComplete" class="complete-message">
      축하합니다! {{ formattedTime }} 만에 완성했습니다!
    </div>

    <div class="buttons">
      <button @click="resetGame" class="btn">새 게임</button>
    </div>

    <div class="preview">
      <p>원본 이미지</p>
      <img :src="imagePath" alt="원본 이미지" />
    </div>
  </div>
</template>

<style scoped>
.puzzle-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: white;
  padding: 30px;
  border-radius: 20px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.stats {
  margin-bottom: 20px;
  font-size: 1.5rem;
  color: #333;
}

.timer {
  background: #f0f0f0;
  padding: 10px 24px;
  border-radius: 25px;
  font-weight: bold;
  font-family: 'Courier New', monospace;
  font-size: 1.8rem;
  letter-spacing: 2px;
  transition: all 0.3s ease;
}

.timer.running {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
}

.timer.completed {
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  color: white;
  box-shadow: 0 4px 15px rgba(56, 239, 125, 0.4);
}

.puzzle-board {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 4px;
  background: #333;
  padding: 4px;
  border-radius: 10px;
  width: 400px;
  height: 400px;
}

.tile {
  position: relative;
  background-color: #f0f0f0;
  border-radius: 6px;
  cursor: default;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.tile.clickable {
  cursor: pointer;
  box-shadow: 0 0 10px rgba(102, 126, 234, 0.5);
}

.tile.clickable:hover {
  transform: scale(0.95);
  box-shadow: 0 0 15px rgba(102, 126, 234, 0.8);
}

.tile.empty {
  background: #222;
  cursor: default;
}

.complete-message {
  margin-top: 20px;
  padding: 15px 30px;
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  color: white;
  border-radius: 10px;
  font-size: 1.2rem;
  font-weight: bold;
  animation: celebrate 0.5s ease;
}

@keyframes celebrate {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}

.buttons {
  margin-top: 20px;
}

.btn {
  padding: 12px 30px;
  font-size: 1rem;
  font-weight: bold;
  color: white;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 20px rgba(102, 126, 234, 0.4);
}

.btn:active {
  transform: translateY(0);
}

.preview {
  margin-top: 30px;
  text-align: center;
}

.preview p {
  color: #666;
  margin-bottom: 10px;
  font-size: 0.9rem;
}

.preview img {
  width: 150px;
  height: 150px;
  object-fit: cover;
  border-radius: 10px;
  border: 3px solid #eee;
}

@media (max-width: 480px) {
  .puzzle-board {
    width: 300px;
    height: 300px;
  }

  .puzzle-container {
    padding: 20px;
  }
}
</style>
