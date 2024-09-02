<script setup lang="ts" generic="T extends any, O extends any">
interface BlockState {
  x: number
  y: number
  revealed: boolean
  mine?: boolean
  flagged: boolean
  adjacentMines: number
}

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

const state: BlockState[][] = reactive(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from({ length: WIDTH }, (_, x) => ({
      x,
      y,
      adjacentMines: 0,
      revealed: false,
      flagged: false,
    }))),
)

let gameover = false
let start = false
let res = ''
function generateMines(initial: BlockState) {
  for (const row of state) {
    for (const block of row) {
      if (Math.abs(block.x - initial.x) < 2 && Math.abs(block.y - initial.y) < 2)
        continue
      block.mine = Math.random() < 0.13
    }
  }

  updateNumbers()
}

function updateNumbers() {
  state.forEach((row) => {
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
  if (state.every(row => row.every(item => item.mine || item.revealed))) {
    res = 'U Win'
    gameover = true
  }
}

function getBlockClass(block: BlockState) {
  if (block.flagged)
    return 'bg-red-500/20'
  if (!gameover && !block.revealed)
    return 'bg-gray-500/20'
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

    return state[ny][nx]
  })
    .filter(Boolean) as BlockState[]
}
</script>

<template>
  <div>
    Mine Sweeper
    <div p5>
      <div class="text-red-500">
        {{ res }}
      </div>
      <div
        v-for="row, y in state"
        :key="y"
        flex="~"
        items-center justify-center
      >
        <button
          v-for="item, x in row"
          :key="x"
          flex="~"

          hover="bg-cyan/20"
          m="0.3"
          h-10
          w-10
          items-center justify-center border :class="getBlockClass(item)" @click="onClick(item)"
          @contextmenu.prevent="onRightClick(item)"
        >
          <template v-if="item.flagged">
            <div i-arcticons:3dmark />
          </template>
          <template v-else-if="gameover || item.revealed">
            <div v-if="item.mine" i-arcticons:bombsquad />
            <div v-else>
              {{ item.adjacentMines }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>
