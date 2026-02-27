<template>
  <NuxtLayout>
    <v-app>
      <v-app-bar
        :elevation="5"
        color="teal-darken-4"
      >
        <template #prepend>
          <v-app-bar-nav-icon />
        </template>
        <v-app-bar-title>
          Minesweeper
        </v-app-bar-title>
      </v-app-bar>
      <v-main>
        <div class="d-flex justify-center m-8">
          <div class="info">
            <b>
              How To Play:
            </b>
            <h2>
              Safe squares have numbers telling you how many mines touch the square.<br>
              You can use the number clues to solve the game by opening all of the safe squares.<br>
              If you click on a mine you lose the game!<br>
            </h2>
          </div>
        </div>
        <div class="d-flex justify-center mt-10 gap-10">
          <div class="mb-2 font-bold">
            <v-icon icon="mdi-flag-variant"/> Flags Left: {{ flagsLeft }}
          </div>
          <div class="mb-2 font-bold">
            Revealed : {{ revealedCount }}
          </div>
          <div class="font-bold mb-2">
            <v-icon icon="mdi-clock-outline"/> Time: {{ time }}s
          </div>
        </div>
        <v-container class="d-flex justify-center mt-10">
          <div class="mr-24">
            <h1>Leaderboard</h1>
            <div v-if="error">
              Error loading data
            </div>
            <ul v-else>
              <li v-for="scoreBoard in scores" :key="scoreBoard.id">
                {{ scoreBoard.username }} - {{ scoreBoard.score }}
              </li>
            </ul>
          </div>
          <div class="board">
            <div
              v-for="(cell, index) in cells"
              :key="index"
              class="cell"
              :class="{
                revealed: cell.isRevealed,
                bomb: cell.isRevealed && cell.isMine,
                flag: cell.isFlagged
               }"
              @click="revealCell(index)"
              @contextmenu.prevent="toggleFlag(index)"
            >
              <span v-if="cell.isRevealed && !cell.isMine && cell.adjacent > 0">
                {{ cell.adjacent }}
              </span>
              <span v-if="cell.isRevealed && cell.isMine"><v-icon icon="mdi-bomb"/></span>
              <span v-if="!cell.isRevealed && cell.isFlagged"><v-icon icon="mdi-flag-variant"/></span>
            </div>
          </div>
          <GameOverDialog
            v-model="showGameOver"
            @restart="restartGame"
          />
          <WinDialog
            v-model="showGameWin"
            :revealedCount="revealedCount"
            :time="time"
            @restart="restartGame"
            @record="handleRecord"
          />
          <v-snackbar
            v-model="showNoFlagWarning"
            timeout="2000"
            color="red"
          >
            No Flag left
          </v-snackbar>
          <div class="d-flex flex-col ml-24 gap-5">
            <div>Revealed Streak: {{ streakCount }}</div>
            <div>Power Up:</div>
            <v-btn
              :disabled="!canUseSafeReveal"
              @click="useSafeReveal"
            >
              <v-icon icon="mdi-magnify"/>Safe Reveal (Streak 5)
            </v-btn>
            <v-btn
              :disabled="!canUseShield"
              @click="useShield"
            >
              <v-icon icon="mdi-shield"/> Shield (Streak 8)
            </v-btn>
            <v-btn
              :disabled="!canUseAutoFlag"
              @click="useAutoFlag"
            >
              <v-icon icon="mdi-flag-variant"/> AutoFlag (Streak 12)
            </v-btn>
          </div>
        </v-container>
      </v-main>
    </v-app>
  </NuxtLayout>
</template>

<script setup lang="ts">
import { onMounted,ref } from 'vue'
import WinDialog from '@/components/game_win.vue'
import GameOverDialog from '@/components/game_over.vue'

const rows = 13
const cols = 6
const totalMines = 10
const streakCount = ref(0)
const flagCount = ref(0)
const revealedCount = ref(0)
const showNoFlagWarning = ref(false)
const flagsLeft = computed(() => totalMines - flagCount.value)
const time = ref(0)
const timer = ref<number | null>(null)
const isGameStarted = ref(false)

type Cell = {
  isMine: boolean
  isRevealed: boolean
  isFlagged: boolean
  adjacent: number
}

function startTimer() {
  if (timer.value !== null) return

  timer.value = setInterval(() => {
    time.value++
  }, 1000)
}
interface Score {
  id: number
  username: string
  score: number
  created_at: string
}
const scores = ref<Score[]>([])

async function loadScores() {
  try {
    scores.value = await $fetch<Score[]>(
      "http://localhost:5000/api/scores"
    )
  } catch (err: any) {
    err.value = err.message
  }
}
function stopTimer() {
  if (timer.value !== null) {
    clearInterval(timer.value)
    timer.value = null
  }
}
async function handleRecord(username: string) {
  try {
    await $fetch("http://localhost:5000/api/scores", {
      method: "POST",
      body: {
        username,
        revealedCell: revealedCount.value,
        time: time.value
      }
    })
    loadScores()
    restartGame()
  } catch (err) {
    console.error(err)
  }
}
function createBoard(): Cell[] {
  const board: Cell[] = Array(rows * cols).fill(null).map(() => ({
    isMine: false,
    isRevealed: false,
    isFlagged: false,
    adjacent: 0,
  }))
  let minesPlaced = 0
  while (minesPlaced < totalMines) {
    const randomIndex = Math.floor(Math.random() * board.length)

    if (!board[randomIndex]!.isMine) {
      board[randomIndex]!.isMine = true
      minesPlaced++
    }
  }
  for (let i = 0; i < board.length; i++) {
    if (board[i]!.isMine) continue

    const row = Math.floor(i / cols)
    const col = i % cols

    let count = 0

    for (let r = -1; r <= 1; r++) {
      for (let c = -1; c <= 1; c++) {
        if (r === 0 && c === 0) continue

        const newRow = row + r
        const newCol = col + c

        if (
          newRow >= 0 &&
          newRow < rows &&
          newCol >= 0 &&
          newCol < cols
        ) {
          const neighborIndex = newRow * cols + newCol
          if (board[neighborIndex]!.isMine) {
            count++
          }
        }
      }
    }

    board[i]!.adjacent = count
  }
  return board
}
function toggleFlag(index: number) {
  const cell = cells.value[index];
  if (!cell || cell.isRevealed) return;

  if(!cell.isFlagged) {
    if(flagCount.value >= totalMines) {
      showNoFlagWarning.value = true
      return
    }
    cell.isFlagged = true
    flagCount.value++
  }
  else {
    cell.isFlagged = false
    flagCount.value--
  }
  checkWin()
}

const showGameOver = ref(false)
const showGameWin = ref(false)

function revealCell(index: number) {
  const cell = cells.value[index]
  if (!cell || cell.isRevealed || cell.isFlagged) return

  if (!isGameStarted.value) {
    isGameStarted.value = true
    startTimer()
  }
  cell.isRevealed = true
  streakCount.value++
  revealedCount.value++
  if (cell.isMine) {
    if (hasShield.value) {
      hasShield.value = false
      return
    }
    stopTimer()
    showGameOver.value = true
    return
  }
  if (cell.adjacent === 0) {
    floodFill(index)
  }
  checkWin()
}
function floodFill(index: number) {
  const row = Math.floor(index / cols)
  const col = index % cols

  for (let r = -1; r <= 1; r++) {
    for (let c = -1; c <= 1; c++) {
      if (r === 0 && c === 0) continue

      const newRow = row + r
      const newCol = col + c

      if (
        newRow >= 0 &&
        newRow < rows &&
        newCol >= 0 &&
        newCol < cols
      ) {
        const neighborIndex = newRow * cols + newCol
        const neighbor = cells.value[neighborIndex]

        if (
          neighbor &&
          !neighbor.isRevealed &&
          !neighbor.isMine
        ) {
          neighbor.isRevealed = true

          if (neighbor.adjacent === 0) {
            floodFill(neighborIndex)
          }
        }
      }
    }
  }
}
function checkWin() {
  if (flagCount.value !== totalMines) return

  const allCorrect = cells.value.every(cell => {
    if (!cell.isMine) {
      return cell.isRevealed
    }
    return true
  })

  if (allCorrect) {
    stopTimer()
    showGameWin.value = true
  }
}
const safeRevealLeft = ref(1)
const shieldLeft = ref(1)
const autoFlagLeft = ref(1)

const hasShield = ref(false)

const canUseSafeReveal = computed(() =>
  streakCount.value >= 5 && safeRevealLeft.value > 0
)

const canUseShield = computed(() =>
  streakCount.value >= 8 && shieldLeft.value > 0
)

const canUseAutoFlag = computed(() =>
  streakCount.value >= 12 && autoFlagLeft.value > 0
)

function useSafeReveal() {
  if (!canUseSafeReveal.value) return
  const safeCells = cells.value
    .map((cell, index) => ({ cell, index }))
    .filter(item => !item.cell.isMine && !item.cell.isRevealed)

  if (!safeCells.length) return

  const random = safeCells[Math.floor(Math.random() * safeCells.length)]
  revealCell(random!.index)
  streakCount.value = streakCount.value - 5
  safeRevealLeft.value--
}

function useShield() {
  if (!canUseShield.value) return
  hasShield.value = true
  streakCount.value = streakCount.value - 8
}
function useAutoFlag() {
  const hiddenMines = cells.value
   .map((cell, index) => ({ cell, index }))
   .filter(item => item.cell.isMine && !item.cell.isFlagged)

  if (!hiddenMines.length) return

  const random = hiddenMines[Math.floor(Math.random() * hiddenMines.length)]

  random!.cell.isFlagged = true
  flagCount.value++

  autoFlagLeft.value--
}

const cells = ref<Cell[]>(createBoard())
function restartGame() {
  stopTimer()
  cells.value = createBoard()
  time.value = 0
  flagCount.value = 0
  revealedCount.value = 0
  streakCount.value = 0
  safeRevealLeft.value = 1
  shieldLeft.value = 1
  autoFlagLeft.value = 1
  showGameOver.value = false 
  showGameWin.value = false
}
onMounted(() => {
  loadScores()
})

</script>

<style scoped>
.board {
  display: grid;
  grid-template-columns: repeat(6, 40px);
  gap: 4px;
}

.info{
  padding: 20px;
  background-color: rgb(255, 235, 122);
  border: 2px solid darkgoldenrod;
  border-radius: 8px;
  color: darkgoldenrod;
}

.cell {
  width: 40px;
  height: 40px;
  background-color: #bdbdbd;
  border: 2px solid #9e9e9e;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-weight: bold;
}

.cell.revealed {
  background-color: #e0e0e0;
  border: 1px solid #9e9e9e;
}

.cell.bomb {
  background: red;
}

.cell:hover {
  background-color: #e0e0e0;
}
</style>
