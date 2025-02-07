<template>
  <div class="game">
    <header class="game-header">
      <h2>{{ gameTitle }}</h2>
      <button @click="router.push('/')" class="back-button">返回主页</button>
    </header>
    <div class="game-container">
      <div class="game-board">
        <!-- 根据游戏类型显示对应的棋盘 -->
        <GoBoard
          v-if="gameType === 'go'"
          ref="goBoard"
          @stone-placed="handleStonePlaced"
          @stones-captured="handleStonesCaptured"
        />
        <ChessBoard
          v-else
          @piece-moved="handlePieceMoved"
        />
      </div>
      <div class="game-sidebar">
        <GoScoreBoard
          v-if="gameType === 'go'"
          ref="goScoreBoard"
          @counting-mode-changed="handleCountingModeChanged"
        />
        <div class="ai-selection">
          <h3>选择AI模型</h3>
          <!-- AI选择组件将在这里渲染 -->
        </div>
        <div class="chat-window">
          <h3>AI对话</h3>
          <!-- 聊天窗口将在这里渲染 -->
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import GoBoard from '@/components/go/GoBoard.vue'
import ChessBoard from '@/components/chess/ChessBoard.vue'
import GoScoreBoard from '@/components/go/GoScoreBoard.vue'

const router = useRouter()
const route = useRoute()

const gameType = computed(() => route.query.type as string)
const gameTitle = computed(() => gameType.value === 'chess' ? '国际象棋' : '围棋')

// 添加记分板引用
const goScoreBoard = ref<InstanceType<typeof GoScoreBoard> | null>(null)

// 添加棋盘引用
const goBoard = ref<InstanceType<typeof GoBoard> | null>(null)

// 修改处理围棋落子事件
const handleStonePlaced = (stone: any) => {
  console.log('Stone placed:', stone)
}

// 修改处理提子事件
const handleStonesCaptured = (data: {
  color: 'black' | 'white'
  count: number
  blackTotal: number
  whiteTotal: number
}) => {
  if (goScoreBoard.value) {
    // 更新双方提子数
    goScoreBoard.value.updateCaptures('black', data.blackTotal)
    goScoreBoard.value.updateCaptures('white', data.whiteTotal)
  }
}

// 处理国际象棋移动事件
const handlePieceMoved = (move: any) => {
  console.log('Piece moved:', move)
  // TODO: 处理移动逻辑，与AI交互
}

// 修改数子模式处理函数
const handleCountingModeChanged = (isCountingMode: boolean) => {
  if (goBoard.value) {
    goBoard.value.setCountingMode(isCountingMode)
    // 重置记分板的领地计数
    if (goScoreBoard.value) {
      goScoreBoard.value.updateTerritory('black', 0)
      goScoreBoard.value.updateTerritory('white', 0)
    }
  }
}
</script>

<style scoped>
.game {
  height: 100%;
  padding: 1rem;
}

.game-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.back-button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background-color: #4a5568;
  color: white;
  cursor: pointer;
}

.game-container {
  display: flex;
  gap: 1rem;
  height: calc(100% - 4rem);
}

.game-board {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: #fff;
  padding: 1rem;
}

.game-sidebar {
  width: 300px;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.ai-selection,
.chat-window {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1rem;
  background-color: #fff;
}

.chat-window {
  flex: 1;
}
</style> 