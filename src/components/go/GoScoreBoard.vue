<template>
  <div class="score-board">
    <div class="score-header">
      <h3>比分</h3>
      <button @click="toggleCountingMode" :class="{ active: isCountingMode }">
        {{ isCountingMode ? '结束数子' : '开始数子' }}
      </button>
    </div>
    
    <div class="scores">
      <div class="player black">
        <div class="player-info">
          <div class="stone black"></div>
          <span>黑方</span>
        </div>
        <div class="score-details">
          <div>领地：{{ hasStarted ? blackTerritory : '-' }}目</div>
          <div>提子：{{ hasStarted ? blackCaptures : '-' }}子</div>
          <div class="total">总计：{{ hasStarted ? blackTotal : '-' }}目</div>
        </div>
      </div>
      
      <div class="player white">
        <div class="player-info">
          <div class="stone white"></div>
          <span>白方</span>
        </div>
        <div class="score-details">
          <div>领地：{{ hasStarted ? whiteTerritory : '-' }}目</div>
          <div>提子：{{ hasStarted ? whiteCaptures : '-' }}子</div>
          <div>贴目：{{ isCountingMode ? komi : '-' }}目</div>
          <div class="total">总计：{{ hasStarted ? whiteTotal : '-' }}目</div>
        </div>
      </div>
    </div>

    <div v-if="winner" class="winner-announcement">
      {{ winner === 'black' ? '黑方' : '白方' }}胜 {{ Math.abs(blackTotal - whiteTotal) }}目
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const props = defineProps({
  komi: {
    type: Number,
    default: 6.5
  }
})

const blackTerritory = ref(0)
const whiteTerritory = ref(0)
const blackCaptures = ref(0)
const whiteCaptures = ref(0)
const isCountingMode = ref(false)
const hasStarted = ref(false)

// 计算总分
const blackTotal = computed(() => blackTerritory.value + blackCaptures.value)
const whiteTotal = computed(() => {
  const baseScore = whiteTerritory.value + whiteCaptures.value
  return isCountingMode.value ? baseScore + props.komi : baseScore
})

// 计算胜者
const winner = computed(() => {
  if (!isCountingMode.value) return null
  
  if (blackTotal.value > whiteTotal.value) return 'black'
  if (whiteTotal.value > blackTotal.value) return 'white'
  return null
})

// 切换数子模式
const toggleCountingMode = () => {
  isCountingMode.value = !isCountingMode.value
  emit('counting-mode-changed', isCountingMode.value)
}

// 更新提子数
const updateCaptures = (color: 'black' | 'white', count: number) => {
  hasStarted.value = true // 有提子时说明对局已开始
  if (color === 'black') {
    blackCaptures.value = count
  } else {
    whiteCaptures.value = count
  }
}

// 更新领地
const updateTerritory = (color: 'black' | 'white', count: number) => {
  if (color === 'black') {
    blackTerritory.value = count
  } else {
    whiteTerritory.value = count
  }
}

// 重置分数
const reset = () => {
  blackTerritory.value = 0
  whiteTerritory.value = 0
  blackCaptures.value = 0
  whiteCaptures.value = 0
  isCountingMode.value = false
  hasStarted.value = false
}

// 定义事件
const emit = defineEmits<{
  (e: 'counting-mode-changed', value: boolean): void
}>()

// 暴露方法给父组件
defineExpose({
  updateCaptures,
  updateTerritory,
  reset
})
</script>

<style scoped>
.score-board {
  background-color: #f5f5f5;
  border-radius: 8px;
  padding: 1rem;
  width: 100%;
}

.score-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.score-header button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background-color: #4a5568;
  color: white;
  cursor: pointer;
  transition: background-color 0.2s;
}

.score-header button.active {
  background-color: #48bb78;
}

.scores {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.player {
  padding: 0.5rem;
  border-radius: 4px;
  background-color: white;
}

.player-info {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
}

.stone {
  width: 20px;
  height: 20px;
  border-radius: 50%;
}

.stone.black {
  background: radial-gradient(circle at 30% 30%, #666, #000);
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.4);
}

.stone.white {
  background: radial-gradient(circle at 30% 30%, #fff, #ddd);
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
  border: 1px solid #ccc;
}

.score-details {
  padding-left: 1.5rem;
}

.total {
  margin-top: 0.5rem;
  font-weight: bold;
}

.winner-announcement {
  margin-top: 1rem;
  text-align: center;
  font-weight: bold;
  color: #2c5282;
  padding: 0.5rem;
  background-color: #ebf8ff;
  border-radius: 4px;
}
</style> 