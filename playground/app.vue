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
        <v-container class="d-flex justify-center mt-10">
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
              <span v-if="cell.isRevealed && cell.isMine">💣</span>
              <span v-if="!cell.isRevealed && cell.isFlagged">🚩</span>
            </div>
          </div>
          <GameOverDialog
            v-model="showGameOver"
            @restart="restartGame"
          />
        </v-container>
      </v-main>
    </v-app>
  </NuxtLayout>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import GameOverDialog from '@/components/game_over.vue'

const rows = 13
const cols = 6
const totalMines = 10

type Cell = {
  isMine: boolean
  isRevealed: boolean
  isFlagged: boolean
  adjacent: number
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

  cell.isFlagged = !cell.isFlagged;
}

const showGameOver = ref(false)

function revealCell(index: number) {
  const cell = cells.value[index]
  if (!cell || cell.isRevealed || cell.isFlagged) return

  cell.isRevealed = true
  if (cell.isMine) {
    showGameOver.value = true
    return
  }
  if (cell.adjacent === 0) {
    floodFill(index)
  }
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
const cells = ref<Cell[]>(createBoard())
function restartGame() {
  cells.value = createBoard()
}

</script>

<style scoped>
.board {
  display: grid;
  grid-template-columns: repeat(6, 40px);
  gap: 4px;
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
