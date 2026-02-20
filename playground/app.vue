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
              :class="{ mine: cell.isMine }"
            >
              {{ cell.isMine ? '💣' : '' }}
            </div>
          </div>
        </v-container>
      </v-main>
    </v-app>
  </NuxtLayout>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const rows = 13
const cols = 6
const totalMines = 6

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

  // สุ่มวางระเบิด
  let minesPlaced = 0
  while (minesPlaced < totalMines) {
    const randomIndex = Math.floor(Math.random() * board.length)

    if (!board[randomIndex]!.isMine) {
      board[randomIndex]!.isMine = true
      minesPlaced++
    }
  }

  return board
}

const cells = ref<Cell[]>(createBoard())


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
  cursor: pointer;
}

.cell:hover {
  background-color: #e0e0e0;
}
</style>
