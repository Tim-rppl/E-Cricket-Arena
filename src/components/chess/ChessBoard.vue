<template>
  <div class="chess-board" :style="boardStyle">
    <!-- 棋盘格子 -->
    <div class="board-squares">
      <div
        v-for="row in 8"
        :key="row"
        class="board-row"
      >
        <div
          v-for="col in 8"
          :key="col"
          class="square"
          :class="{
            'white': isWhiteSquare(row, col),
            'black': !isWhiteSquare(row, col),
            'selected': isSelected(row, col),
            'valid-move': isValidMove(row, col)
          }"
          @click="handleSquareClick(row, col)"
        >
          <!-- 棋子 -->
          <div
            v-if="getPiece(row, col)"
            class="piece"
            :class="[
              getPiece(row, col)?.color,
              getPiece(row, col)?.type
            ]"
          ></div>
        </div>
      </div>
    </div>
    <!-- 坐标标注 -->
    <div class="coordinates">
      <div class="files">
        <span v-for="file in files" :key="file">{{ file }}</span>
      </div>
      <div class="ranks">
        <span v-for="rank in ranks" :key="rank">{{ rank }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

// 棋子类型定义
interface ChessPiece {
  type: 'pawn' | 'rook' | 'knight' | 'bishop' | 'queen' | 'king'
  color: 'white' | 'black'
  position: { row: number; col: number }
}

// 棋盘属性
const props = defineProps({
  squareSize: {
    type: Number,
    default: 60 // 每个格子的大小（像素）
  }
})

// 棋盘状态
const selectedPiece = ref<ChessPiece | null>(null)
const pieces = ref<ChessPiece[]>([])
const files = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
const ranks = ['8', '7', '6', '5', '4', '3', '2', '1']

// 计算棋盘大小
const boardSize = computed(() => props.squareSize * 8)
const boardStyle = computed(() => ({
  width: `${boardSize.value}px`,
  height: `${boardSize.value}px`
}))

// 初始化棋盘
const initializeBoard = () => {
  // 添加白方棋子
  pieces.value = [
    // 白方棋子
    { type: 'rook', color: 'white', position: { row: 8, col: 1 } },
    { type: 'knight', color: 'white', position: { row: 8, col: 2 } },
    { type: 'bishop', color: 'white', position: { row: 8, col: 3 } },
    { type: 'queen', color: 'white', position: { row: 8, col: 4 } },
    { type: 'king', color: 'white', position: { row: 8, col: 5 } },
    { type: 'bishop', color: 'white', position: { row: 8, col: 6 } },
    { type: 'knight', color: 'white', position: { row: 8, col: 7 } },
    { type: 'rook', color: 'white', position: { row: 8, col: 8 } },
    // 白方兵
    ...Array.from({ length: 8 }, (_, i) => ({
      type: 'pawn' as const,
      color: 'white' as const,
      position: { row: 7, col: i + 1 }
    })),
    // 黑方棋子
    { type: 'rook', color: 'black', position: { row: 1, col: 1 } },
    { type: 'knight', color: 'black', position: { row: 1, col: 2 } },
    { type: 'bishop', color: 'black', position: { row: 1, col: 3 } },
    { type: 'queen', color: 'black', position: { row: 1, col: 4 } },
    { type: 'king', color: 'black', position: { row: 1, col: 5 } },
    { type: 'bishop', color: 'black', position: { row: 1, col: 6 } },
    { type: 'knight', color: 'black', position: { row: 1, col: 7 } },
    { type: 'rook', color: 'black', position: { row: 1, col: 8 } },
    // 黑方兵
    ...Array.from({ length: 8 }, (_, i) => ({
      type: 'pawn' as const,
      color: 'black' as const,
      position: { row: 2, col: i + 1 }
    }))
  ]
}

// 判断格子颜色
const isWhiteSquare = (row: number, col: number) => {
  return (row + col) % 2 === 0
}

// 获取格子上的棋子
const getPiece = (row: number, col: number) => {
  return pieces.value.find(p => 
    p.position.row === row && p.position.col === col
  )
}

// 判断是否选中
const isSelected = (row: number, col: number) => {
  return selectedPiece.value?.position.row === row && 
         selectedPiece.value?.position.col === col
}

// 判断是否为有效移动位置
const isValidMove = (row: number, col: number) => {
  // TODO: 实现移动规则验证
  return false
}

// 处理格子点击
const handleSquareClick = (row: number, col: number) => {
  const piece = getPiece(row, col)
  
  if (selectedPiece.value) {
    if (piece?.color === selectedPiece.value.color) {
      // 选择同色棋子，更新选中
      selectedPiece.value = piece
    } else {
      // 移动棋子
      movePiece(selectedPiece.value, row, col)
      selectedPiece.value = null
    }
  } else if (piece) {
    selectedPiece.value = piece
  }
}

// 移动棋子
const movePiece = (piece: ChessPiece, toRow: number, toCol: number) => {
  // TODO: 验证移动是否合法
  const index = pieces.value.indexOf(piece)
  if (index !== -1) {
    // 移除目标位置的棋子（吃子）
    pieces.value = pieces.value.filter(p => 
      !(p.position.row === toRow && p.position.col === toCol)
    )
    // 更新棋子位置
    pieces.value[index].position = { row: toRow, col: toCol }
    // 触发移动事件
    emit('piece-moved', {
      piece,
      from: { row: piece.position.row, col: piece.position.col },
      to: { row: toRow, col: toCol }
    })
  }
}

// 定义事件
const emit = defineEmits<{
  (e: 'piece-moved', move: {
    piece: ChessPiece,
    from: { row: number, col: number },
    to: { row: number, col: number }
  }): void
}>()

// 初始化棋盘
initializeBoard()
</script>

<style scoped>
.chess-board {
  position: relative;
  border: 2px solid #654321;
  margin: 20px;
}

.board-squares {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.board-row {
  display: flex;
  flex: 1;
}

.square {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

.square.white {
  background-color: #f0d9b5;
}

.square.black {
  background-color: #b58863;
}

.square.selected {
  background-color: #7b61ff;
}

.square.valid-move::before {
  content: '';
  position: absolute;
  width: 20%;
  height: 20%;
  border-radius: 50%;
  background-color: rgba(0, 255, 0, 0.3);
}

.piece {
  width: 80%;
  height: 80%;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}

/* 棋子样式 - 使用SVG或图片 */
.piece.white.king {
  background-image: url('@/assets/images/chess/white-king.svg');
}
.piece.white.queen {
  background-image: url('@/assets/images/chess/white-queen.svg');
}
/* ... 其他棋子的背景图片 ... */

.coordinates {
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.files {
  position: absolute;
  bottom: -20px;
  left: 0;
  right: 0;
  display: flex;
  justify-content: space-around;
}

.ranks {
  position: absolute;
  top: 0;
  bottom: 0;
  left: -20px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}

.files span, .ranks span {
  font-size: 12px;
  color: #666;
}
</style> 