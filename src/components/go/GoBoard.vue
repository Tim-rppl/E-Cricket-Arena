<template>
  <div 
    class="go-board" 
    :style="boardStyle"
    @click="handleClick"
    @mousemove="handleMouseMove"
    @mouseleave="handleMouseLeave"
  >
    <!-- 棋盘网格 -->
    <canvas 
      ref="gridCanvas" 
      class="board-grid"
      :width="boardSize" 
      :height="boardSize"
    ></canvas>
    <!-- 棋子层 -->
    <div class="stones-layer">
      <div
        v-for="stone in stones"
        :key="`${stone.x}-${stone.y}`"
        class="stone"
        :class="stone.color"
        :style="getStoneStyle(stone)"
      ></div>
      <!-- 预览提示 -->
      <div
        v-if="previewStone"
        class="stone preview"
        :class="[currentColor, { 'invalid': !isValidMove(previewStone.x, previewStone.y) }]"
        :style="getStoneStyle(previewStone)"
      ></div>
    </div>
    <!-- 最后一手标记 -->
    <div
      v-if="lastMove"
      class="last-move-marker"
      :style="getMarkerStyle(lastMove)"
    ></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'

// 定义棋盘属性
const props = defineProps({
  size: {
    type: Number,
    default: 19 // 标准围棋棋盘为19路
  },
  cellSize: {
    type: Number,
    default: 30 // 每个格子的大小（像素）
  }
})

// 棋盘状态
interface Stone {
  x: number
  y: number
  color: 'black' | 'white'
}

interface Position {
  x: number
  y: number
}

const stones = ref<Stone[]>([])
const lastMove = ref<Stone | null>(null)
const gridCanvas = ref<HTMLCanvasElement | null>(null)
const previewStone = ref<Stone | null>(null)
const currentColor = computed(() => stones.value.length % 2 === 0 ? 'black' : 'white')
const koPoint = ref<Position | null>(null)

// 在 script setup 中添加提子计数
const blackCaptures = ref(0)
const whiteCaptures = ref(0)
const isCountingMode = ref(false)

// 计算棋盘大小
const boardSize = computed(() => props.cellSize * (props.size - 1))
const boardStyle = computed(() => ({
  width: `${boardSize.value}px`,
  height: `${boardSize.value}px`
}))

// 获取某个位置的棋子
const getStone = (x: number, y: number) => {
  return stones.value.find(s => s.x === x && s.y === y)
}

// 获取某个位置的相邻点
const getAdjacent = (x: number, y: number): Position[] => {
  const adjacent: Position[] = []
  if (x > 0) adjacent.push({ x: x - 1, y })
  if (x < props.size - 1) adjacent.push({ x: x + 1, y })
  if (y > 0) adjacent.push({ x, y: y - 1 })
  if (y < props.size - 1) adjacent.push({ x, y: y + 1 })
  return adjacent
}

// 修改原有的 getLiberties 函数
const getLiberties = (group: Position[]): Position[] => {
  const liberties = new Set<string>()
  
  group.forEach(pos => {
    getAdjacent(pos.x, pos.y).forEach(adj => {
      // 检查这个位置是否已经有棋子
      const stone = getStone(adj.x, adj.y)
      if (!stone) {
        liberties.add(`${adj.x},${adj.y}`)
      }
    })
  })

  return Array.from(liberties).map(str => {
    const [x, y] = str.split(',').map(Number)
    return { x, y }
  })
}

// 获取一个棋子所在的整个棋串
const getGroup = (stone: Stone): Position[] => {
  const group = new Set<string>()
  const toCheck = [`${stone.x},${stone.y}`]
  
  while (toCheck.length > 0) {
    const pos = toCheck.pop()!
    if (group.has(pos)) continue
    
    const [x, y] = pos.split(',').map(Number)
    group.add(pos)
    
    getAdjacent(x, y).forEach(adj => {
      const adjStone = getStone(adj.x, adj.y)
      if (adjStone && adjStone.color === stone.color) {
        const adjPos = `${adj.x},${adj.y}`
        if (!group.has(adjPos)) {
          toCheck.push(adjPos)
        }
      }
    })
  }
  
  return Array.from(group).map(str => {
    const [x, y] = str.split(',').map(Number)
    return { x, y }
  })
}

// 修改事件定义
const emit = defineEmits<{
  (e: 'stone-placed', stone: Stone): void
  (e: 'stones-captured', data: {
    color: 'black' | 'white'
    count: number
    blackTotal: number
    whiteTotal: number
  }): void
}>()

// 修改提子函数
const captureDeadStones = (x: number, y: number, color: 'black' | 'white') => {
  const capturedStones: Position[] = []
  
  // 检查所有相邻的对方棋子
  getAdjacent(x, y).forEach(adj => {
    const stone = getStone(adj.x, adj.y)
    if (stone && stone.color !== color) {
      const group = getGroup(stone)
      // 计算这个群的气
      const liberties = getLiberties(group)
      // 如果这个群没气了，就提掉
      if (liberties.length === 0) {
        capturedStones.push(...group)
      }
    }
  })
  
  // 如果只提了一个子，记录打劫点
  if (capturedStones.length === 1) {
    koPoint.value = capturedStones[0]
  } else {
    koPoint.value = null
  }
  
  // 如果有提子，更新棋盘状态
  if (capturedStones.length > 0) {
    // 更新棋盘状态
    stones.value = stones.value.filter(stone => 
      !capturedStones.some(pos => pos.x === stone.x && pos.y === stone.y)
    )
    
    // 更新提子计数
    if (color === 'black') {
      blackCaptures.value += capturedStones.length
    } else {
      whiteCaptures.value += capturedStones.length
    }
    
    // 触发提子事件
    emit('stones-captured', {
      color,
      count: capturedStones.length,
      blackTotal: blackCaptures.value,
      whiteTotal: whiteCaptures.value
    })
  }
  
  return capturedStones.length
}

// 修改放置棋子的函数
const placeStone = (x: number, y: number) => {
  if (!isValidMove(x, y)) return
  
  const color = currentColor.value
  const newStone: Stone = { x, y, color }
  stones.value.push(newStone)
  
  // 先检查是否提掉对方的子
  const capturedCount = captureDeadStones(x, y, color)
  
  // 如果没有提子，且这步棋自己没气了，说明是自杀，撤销这步棋
  if (capturedCount === 0) {
    const group = getGroup(newStone)
    if (getLiberties(group).length === 0) {
      stones.value.pop() // 撤销这步棋
      return
    }
  }
  
  lastMove.value = newStone
  emit('stone-placed', newStone)
}

// 修改判断是否为有效移动的函数
const isValidMove = (x: number, y: number) => {
  // 检查是否已有棋子
  if (getStone(x, y)) return false
  
  // 检查打劫
  if (koPoint.value && koPoint.value.x === x && koPoint.value.y === y) {
    return false
  }
  
  const color = currentColor.value
  // 模拟落子
  const testStone: Stone = { x, y, color }
  
  // 临时添加这个棋子来检查状态
  stones.value.push(testStone)
  
  // 检查这步棋落下后的状态
  const group = getGroup(testStone)
  const hasLiberties = getLiberties(group).length > 0
  
  // 检查是否能提掉对方的子
  let canCapture = false
  getAdjacent(x, y).forEach(adj => {
    const stone = getStone(adj.x, adj.y)
    if (stone && stone.color !== color) {
      const enemyGroup = getGroup(stone)
      if (getLiberties(enemyGroup).length === 0) {
        canCapture = true
      }
    }
  })
  
  // 移除临时添加的棋子
  stones.value.pop()
  
  // 如果这步棋有气，或者能提子，就是合法的
  return hasLiberties || canCapture
}

// 修改处理点击事件
const handleClick = (event: MouseEvent) => {
  // 在数子模式下禁用落子
  if (isCountingMode.value) {
    return
  }

  const rect = (event.currentTarget as HTMLElement).getBoundingClientRect()
  const offsetX = event.clientX - rect.left
  const offsetY = event.clientY - rect.top
  
  const x = Math.floor(offsetX / props.cellSize)
  const y = Math.floor(offsetY / props.cellSize)
  
  if (x >= 0 && x < props.size && y >= 0 && y < props.size) {
    placeStone(x, y)
  }
}

// 修改处理鼠标移动事件
const handleMouseMove = (event: MouseEvent) => {
  // 在数子模式下不显示预览
  if (isCountingMode.value) {
    previewStone.value = null
    return
  }

  const rect = (event.currentTarget as HTMLElement).getBoundingClientRect()
  const offsetX = event.clientX - rect.left
  const offsetY = event.clientY - rect.top
  
  const x = Math.floor(offsetX / props.cellSize)
  const y = Math.floor(offsetY / props.cellSize)
  
  if (x >= 0 && x < props.size && y >= 0 && y < props.size) {
    previewStone.value = { x, y, color: currentColor.value }
  } else {
    previewStone.value = null
  }
}

// 处理鼠标离开
const handleMouseLeave = () => {
  previewStone.value = null
}

// 添加设置数子模式的方法
const setCountingMode = (mode: boolean) => {
  isCountingMode.value = mode
}

// 组件挂载时绘制棋盘
onMounted(() => {
  drawGrid()
})

// 绘制棋盘网格
const drawGrid = () => {
  const canvas = gridCanvas.value
  if (!canvas) return

  const ctx = canvas.getContext('2d')
  if (!ctx) return

  ctx.strokeStyle = '#000000'
  ctx.lineWidth = 1

  // 绘制横线和竖线
  for (let i = 0; i < props.size; i++) {
    const position = i * props.cellSize

    // 横线
    ctx.beginPath()
    ctx.moveTo(0, position)
    ctx.lineTo(boardSize.value, position)
    ctx.stroke()

    // 竖线
    ctx.beginPath()
    ctx.moveTo(position, 0)
    ctx.lineTo(position, boardSize.value)
    ctx.stroke()
  }

  // 绘制星位
  const starPoints = getStarPoints()
  ctx.fillStyle = '#000000'
  starPoints.forEach(point => {
    ctx.beginPath()
    ctx.arc(
      point.x * props.cellSize,
      point.y * props.cellSize,
      3,
      0,
      2 * Math.PI
    )
    ctx.fill()
  })
}

// 获取星位坐标
const getStarPoints = () => {
  if (props.size === 19) {
    return [
      { x: 3, y: 3 },
      { x: 9, y: 3 },
      { x: 15, y: 3 },
      { x: 3, y: 9 },
      { x: 9, y: 9 },
      { x: 15, y: 9 },
      { x: 3, y: 15 },
      { x: 9, y: 15 },
      { x: 15, y: 15 }
    ]
  }
  // 可以添加其他规格棋盘的星位
  return []
}

// 计算棋子样式
const getStoneStyle = (stone: Stone) => {
  return {
    left: `${stone.x * props.cellSize - props.cellSize / 2}px`,
    top: `${stone.y * props.cellSize - props.cellSize / 2}px`,
    width: `${props.cellSize - 2}px`,
    height: `${props.cellSize - 2}px`
  }
}

// 计算标记样式
const getMarkerStyle = (stone: Stone) => {
  return {
    left: `${stone.x * props.cellSize - 4}px`,
    top: `${stone.y * props.cellSize - 4}px`
  }
}

// 扩展暴露的属性和方法
defineExpose({
  stones,
  lastMove,
  koPoint,
  blackCaptures,
  whiteCaptures,
  reset: () => {
    stones.value = []
    lastMove.value = null
    koPoint.value = null
    blackCaptures.value = 0
    whiteCaptures.value = 0
    isCountingMode.value = false
  },
  setCountingMode
})
</script>

<style scoped>
.go-board {
  position: relative;
  background-color: #DEB887;
  border: 1px solid #000000;
  margin: 20px;
  cursor: pointer;
  user-select: none;
}

.board-grid {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  pointer-events: none;
}

.stones-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
  pointer-events: none;
}

.stone {
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
  transition: all 0.2s ease;
}

.stone.black {
  background: radial-gradient(circle at 30% 30%, #666, #000);
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.4);
}

.stone.white {
  background: radial-gradient(circle at 30% 30%, #fff, #ddd);
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.last-move-marker {
  position: absolute;
  width: 8px;
  height: 8px;
  background-color: #ff0000;
  border-radius: 50%;
  pointer-events: none;
}

.stone.preview {
  opacity: 0.5;
}

.stone.preview.invalid {
  opacity: 0.3;
  background: #ff0000;
}
</style> 