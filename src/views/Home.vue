<template>
  <div class="home">
    <h1>电子斗蛐蛐</h1>
    <div class="game-selection">
      <h2>选择游戏</h2>
      <div class="game-options">
        <div class="game-option" @click="navigateToGame('chess')">
          <h3>国际象棋</h3>
          <p>体验国际象棋的魅力</p>
        </div>
        <div class="game-option" @click="navigateToGame('go')">
          <h3>围棋</h3>
          <p>感受围棋的深奥</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'

const router = useRouter()

const navigateToGame = (type: 'chess' | 'go') => {
  router.push({
    name: 'game',
    query: { type }
  })
}
</script>

<style scoped>
.home {
  padding: 2rem;
  text-align: center;
}

.game-selection {
  margin-top: 2rem;
}

.game-options {
  display: flex;
  justify-content: center;
  gap: 2rem;
  margin-top: 2rem;
}

.game-option {
  padding: 2rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.game-option:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}
</style> 