<template>
  <div class="go-board" :style="boardStyle">
    <!-- 棋盘网格 -->
    <div class="board-grid">
      <canvas ref="gridCanvas" :width="boardSize" :height="boardSize"></canvas>
    </div>
    <!-- 棋子层 -->
    <div class="stones-layer">
      <div
        v-for="(stone, index) in stones"
        :key="index"
        class="stone"
        :class="stone.color"
        :style="getStoneStyle(stone)"
        @click="placeStone(stone.x, stone.y)"
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

const stones = ref<Stone[]>([])
const lastMove = ref<Stone | null>(null)
const gridCanvas = ref<HTMLCanvasElement | null>(null)

// 计算棋盘大小
const boardSize = computed(() => props.cellSize * (props.size - 1))
const boardStyle = computed(() => ({
  width: `${boardSize.value}px`,
  height: `${boardSize.value}px`
}))

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

// 放置棋子
const placeStone = (x: number, y: number) => {
  // 检查是否已有棋子
  const existingStone = stones.value.find(s => s.x === x && s.y === y)
  if (existingStone) return

  // 确定棋子颜色（这里简单地根据已有棋子数量判断）
  const color = stones.value.length % 2 === 0 ? 'black' : 'white'
  
  const newStone: Stone = { x, y, color }
  stones.value.push(newStone)
  lastMove.value = newStone

  // 触发落子事件
  emit('stone-placed', newStone)
}

// 定义事件
const emit = defineEmits<{
  (e: 'stone-placed', stone: Stone): void
}>()

// 组件挂载时绘制棋盘
onMounted(() => {
  drawGrid()
})
</script>

<style scoped>
.go-board {
  position: relative;
  background-color: #DEB887; /* 实木色背景 */
  border: 1px solid #000000;
  margin: 20px;
}

.board-grid {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.stones-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.stone {
  position: absolute;
  border-radius: 50%;
  cursor: pointer;
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
</style> 