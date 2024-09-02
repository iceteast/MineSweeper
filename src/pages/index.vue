<script setup lang="ts" generic="T extends any, O extends any">
import type { BlockState } from '~/types'

const WIDTH = 10
const HEIGHT = 10
const DIRECTIONS = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
]
const numberColor = [
  'text-transparent',
  'text-red-500',
  'text-yellow-500',
  'text-cyan-500',
  'text-green-500',
  'text-purple-500',
  'text-blue-500',
  'text-pink-500',
  'text-orange-500',
]

const state = ref(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from({ length: WIDTH }, (_, x) => ({
      x,
      y,
      adjacentMines: 0,
      revealed: false,
      mine: false,
      flagged: false,
    }))),
)

let gameover = false
let start = false
let res = ''
let startTime = Date.now()
let time = ''

function restart() {
  gameover = false
  start = false
  res = ''

  for (const row of state.value) {
    for (const block of row) {
      block.adjacentMines = 0
      block.revealed = false
      block.mine = false
      block.flagged = false
    }
  }
}
function generateMines(initial: BlockState) {
  for (const row of state.value) {
    for (const block of row) {
      if (Math.abs(block.x - initial.x) < 2 && Math.abs(block.y - initial.y) < 2)
        continue
      block.mine = Math.random() < 0.13
    }
  }

  updateNumbers()
  startTime = Date.now()
}

function updateNumbers() {
  state.value.forEach((row) => {
    row.forEach((block) => {
      if (block.mine)
        return
      getSiblings(block).forEach((item) => {
        if (item.mine)
          block.adjacentMines += 1
      })
    })
  })
}

function checkWin(block: BlockState) {
  if (block.mine) {
    res = 'BOOM'
    gameover = true
  }
  if (state.value.every(row => row.every(item => item.mine || item.revealed))) {
    res = 'U Win'
    gameover = true
  }
}

function getBlockClass(block: BlockState) {
  if (block.flagged)
    return 'bg-red-500/20'
  if (!gameover && !block.revealed)
    return 'bg-gray-500/20 hover:bg-cyan/20'
  return block.mine ? 'bg-red-500/30' : numberColor[block.adjacentMines]
}
function onClick(block: BlockState) {
  // initialize board
  if (!start) {
    start = true
    generateMines(block)
  }
  // click
  expend(block)
  flip(block)

  // win
  checkWin(block)
}

function flip(block: BlockState) {
  if (block.flagged)
    block.flagged = false
  if (!block.revealed)
    block.revealed = true
}

function onRightClick(block: BlockState) {
  if (!block.revealed)
    block.flagged = !block.flagged
}

function expend(block: BlockState) {
  if (block.adjacentMines)
    return

  getSiblings(block).forEach((b) => {
    if (!b.revealed) {
      flip(b)
      expend(b)
    }
  })
}

function getSiblings(block: BlockState) {
  return DIRECTIONS.map(([dx, dy]) => {
    const nx = block.x + dx
    const ny = block.y + dy
    if (nx < 0 || nx >= WIDTH || ny < 0 || ny >= HEIGHT)
      return undefined

    return state.value[ny][nx]
  })
    .filter(Boolean) as BlockState[]
}

function getStatus() {
  let f = 0
  let b = 0
  for (const row of state.value) {
    for (const block of row) {
      if (block.flagged)
        f++
      if (block.mine)
        b++
    }
  }
  return `${f.toString()}/${b.toString()}`
}

function timer() {
  time = ((Date.now() - startTime) / 1000).toString()
}

setInterval(
  () => { timer() },
  10,
)
</script>

<template>
  <div>
    Mine Sweeper
    <div p5>
      <div v-if="gameover">
        <button
          hover="bg-amber/20"
          w-106
          border
          @click="restart"
        >
          restart
        </button>
      </div>
      <div
        v-else
        align="center"
      >
        <table
          w-106
          border-1
        >
          <tr>
            <td w-10>
              {{ getStatus() }}
            </td>
            <td w-80>
              {{ res }}
            </td>
            <td w-10>
              {{ time }}
            </td>
          </tr>
        </table>
      </div>
      <div
        v-for="row, y in state"
        :key="y"
        flex="~"
        items-center
        justify-center
      >
        <button
          v-for="item, x in row"
          :key="x"
          flex="~"
          m="0.3"
          h-10
          w-10
          items-center
          justify-center
          border
          :class="getBlockClass(item)"
          @click="onClick(item)"
          @contextmenu.prevent="onRightClick(item)"
        >
          <template v-if="item.flagged">
            <div i-arcticons:3dmark />
          </template>
          <template v-else-if="gameover || item.revealed">
            <div
              v-if="item.mine"
              i-arcticons:bombsquad
            />
            <div v-else>
              {{ item.adjacentMines }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>
