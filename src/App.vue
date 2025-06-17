<template>
  <div class="app">
    <h1 class="logo">Bingo</h1>
    <input type="number" v-model="bingoCount" />
    <button @click="newGame">New Game</button>
    <h2 class="lines">
      <b>{{ lines.length }}</b>
      <span>Lines</span>
    </h2>
    <div class="bingo-container">
      <div class="bingo-board">
        <div v-for="num in bingoNumbers" :key="num" class="cell" :class="{ active: activeNumbers.includes(num) }" @click="markNumber(num)">
          {{ num }}
        </div>
      </div>
      <div class="line-layer">
        <div v-for="(line, i) in lines" :key="i" class="line" :class="getLineClass(line)"></div>
      </div>
    </div>
    <div class="called box-call">
      <h3>Called</h3>
      <ul>
        <li v-for="num in calledNumbers" :key="'called-' + num" :class="{ my: bingoNumbers.includes(num) }" @click="handleNum(num)">
          {{ num }}
        </li>
      </ul>
    </div>

    <div class="uncalled box-call">
      <h3>Uncalled</h3>
      <ul>
        <li v-for="num in uncalledNumbers" :key="'uncalled-' + num" :class="{ my: bingoNumbers.includes(num) }" @click="handleNum(num)">
          {{ num }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

const bingoCount = ref(50)
const bingoNumbers = ref<number[]>([])
const activeNumbers = ref<number[]>([])
const calledNumbers = ref<number[]>([])
const uncalledNumbers = computed(() => Array.from({ length: bingoCount.value }, (_, i) => i + 1).filter(n => !calledNumbers.value.includes(n)))
const lines = ref<number[][]>([])

const shuffle = (arr: number[]): number[] => {
  return arr.sort(() => Math.random() - 0.5)
}
const handleNum = (num: number): void => {
  const idx = activeNumbers.value.indexOf(num)
  if (idx === -1) {
    activeNumbers.value.push(num)
    calledNumbers.value.push(num)
  } else {
    activeNumbers.value.splice(idx, 1)
    calledNumbers.value.splice(idx, 1)
  }
  checkBingo()
}
const newGame = (): void => {
  const pool: number[] = Array.from({ length: bingoCount.value }, (_, i) => i + 1)
  bingoNumbers.value = shuffle(pool).slice(0, 25) // 從 pool 隨機取前 25 個
  activeNumbers.value = []
  calledNumbers.value = []
  lines.value = []
}

const markNumber = (num: number): void => {
  const idx = activeNumbers.value.indexOf(num)
  if (idx === -1) {
    activeNumbers.value.push(num)
    calledNumbers.value.push(num)
  } else {
    activeNumbers.value.splice(idx, 1)
    calledNumbers.value.splice(idx, 1)
  }
  checkBingo()
}

const checkBingo = (): void => {
  const board = bingoNumbers.value
  const active = activeNumbers.value
  const newLines: number[][] = []

  const getIdx = (r: number, c: number): number => r * 5 + c
  const isLine = (indexes: number[]): boolean => indexes.every(i => active.includes(board[i]))

  for (let r = 0; r < 5; r++) {
    const row = Array.from({ length: 5 }, (_, c) => getIdx(r, c))
    if (isLine(row)) newLines.push(row)
  }
  for (let c = 0; c < 5; c++) {
    const col = Array.from({ length: 5 }, (_, r) => getIdx(r, c))
    if (isLine(col)) newLines.push(col)
  }
  const diag1 = Array.from({ length: 5 }, (_, i) => getIdx(i, i))
  const diag2 = Array.from({ length: 5 }, (_, i) => getIdx(i, 4 - i))
  if (isLine(diag1)) newLines.push(diag1)
  if (isLine(diag2)) newLines.push(diag2)

  lines.value = newLines
}

const getLineClass = (line: number[]): string => {
  const indexes = line.map(i => i)
  const rowSet = new Set(indexes.map(i => Math.floor(i / 5)))
  const colSet = new Set(indexes.map(i => i % 5))

  if (rowSet.size === 1) return `row-${[...rowSet][0]}`
  if (colSet.size === 1) return `col-${[...colSet][0]}`
  if (indexes.join() === '0,6,12,18,24') return 'diag-main'
  if (indexes.join() === '4,8,12,16,20') return 'diag-anti'
  return ''
}

onMounted(() => {
  newGame()
})
</script>
