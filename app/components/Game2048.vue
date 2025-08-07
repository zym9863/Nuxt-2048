<template>
  <div class="game-container">
    <div class="game-header">
      <h1>2048</h1>
      <div class="score-container">
        <div class="score-box">
          <div class="score-label">分数</div>
          <div class="score">{{ score }}</div>
        </div>
        <div class="score-box">
          <div class="score-label">最高分</div>
          <div class="score">{{ bestScore }}</div>
        </div>
      </div>
    </div>
    
    <div class="game-intro">
      <p>使用方向键移动方块，相同数字的方块会合并！</p>
      <button @click="newGame" class="restart-button">新游戏</button>
    </div>

    <div class="grid-container">
      <div class="grid-row" v-for="(row, rowIndex) in grid" :key="rowIndex">
        <div 
          class="grid-cell" 
          v-for="(cell, colIndex) in row" 
          :key="colIndex"
          :class="getCellClass(cell)"
        >
          <span v-if="cell !== 0">{{ cell }}</span>
        </div>
      </div>
    </div>

    <div v-if="gameOver" class="game-over-overlay">
      <div class="game-over-message">
        <h2>游戏结束!</h2>
        <p>最终分数: {{ score }}</p>
        <button @click="newGame" class="restart-button">再来一局</button>
      </div>
    </div>

    <div v-if="gameWon && !gameOver" class="game-won-overlay">
      <div class="game-won-message">
        <h2>恭喜！你达到了2048!</h2>
        <p>分数: {{ score }}</p>
        <button @click="continueGame" class="continue-button">继续游戏</button>
        <button @click="newGame" class="restart-button">新游戏</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// 游戏状态
const grid = ref([])
const score = ref(0)
const bestScore = ref(0)
const gameOver = ref(false)
const gameWon = ref(false)

/**
 * 初始化游戏网格
 */
const initGrid = () => {
  grid.value = Array(4).fill().map(() => Array(4).fill(0))
}

/**
 * 在随机空位置添加新数字（2或4）
 */
const addRandomTile = () => {
  const emptyCells = []
  for (let i = 0; i < 4; i++) {
    for (let j = 0; j < 4; j++) {
      if (grid.value[i][j] === 0) {
        emptyCells.push({ row: i, col: j })
      }
    }
  }
  
  if (emptyCells.length > 0) {
    const randomCell = emptyCells[Math.floor(Math.random() * emptyCells.length)]
    const value = Math.random() < 0.9 ? 2 : 4
    grid.value[randomCell.row][randomCell.col] = value
  }
}

/**
 * 开始新游戏
 */
const newGame = () => {
  initGrid()
  score.value = 0
  gameOver.value = false
  gameWon.value = false
  addRandomTile()
  addRandomTile()
}

/**
 * 继续游戏（达到2048后）
 */
const continueGame = () => {
  gameWon.value = false
}

/**
 * 获取单元格的CSS类名
 */
const getCellClass = (value) => {
  if (value === 0) return ''
  return `tile-${value}`
}

/**
 * 向左移动
 */
const moveLeft = () => {
  let moved = false
  const newGrid = grid.value.map(row => [...row])
  
  for (let i = 0; i < 4; i++) {
    const row = newGrid[i].filter(val => val !== 0)
    
    // 合并相同的数字
    for (let j = 0; j < row.length - 1; j++) {
      if (row[j] === row[j + 1]) {
        row[j] *= 2
        score.value += row[j]
        row[j + 1] = 0
        
        // 检查是否达到2048
        if (row[j] === 2048 && !gameWon.value) {
          gameWon.value = true
        }
      }
    }
    
    // 移除0并重新排列
    const filteredRow = row.filter(val => val !== 0)
    while (filteredRow.length < 4) {
      filteredRow.push(0)
    }
    
    // 检查是否有移动
    for (let j = 0; j < 4; j++) {
      if (newGrid[i][j] !== filteredRow[j]) {
        moved = true
      }
    }
    
    newGrid[i] = filteredRow
  }
  
  if (moved) {
    grid.value = newGrid
    addRandomTile()
    checkGameOver()
    updateBestScore()
  }
}

/**
 * 向右移动
 */
const moveRight = () => {
  let moved = false
  const newGrid = grid.value.map(row => [...row])
  
  for (let i = 0; i < 4; i++) {
    const row = newGrid[i].filter(val => val !== 0)
    
    // 从右到左合并
    for (let j = row.length - 1; j > 0; j--) {
      if (row[j] === row[j - 1]) {
        row[j] *= 2
        score.value += row[j]
        row[j - 1] = 0
        
        if (row[j] === 2048 && !gameWon.value) {
          gameWon.value = true
        }
      }
    }
    
    const filteredRow = row.filter(val => val !== 0)
    while (filteredRow.length < 4) {
      filteredRow.unshift(0)
    }
    
    for (let j = 0; j < 4; j++) {
      if (newGrid[i][j] !== filteredRow[j]) {
        moved = true
      }
    }
    
    newGrid[i] = filteredRow
  }
  
  if (moved) {
    grid.value = newGrid
    addRandomTile()
    checkGameOver()
    updateBestScore()
  }
}

/**
 * 向上移动
 */
const moveUp = () => {
  let moved = false
  const newGrid = grid.value.map(row => [...row])
  
  for (let j = 0; j < 4; j++) {
    const column = []
    for (let i = 0; i < 4; i++) {
      if (newGrid[i][j] !== 0) {
        column.push(newGrid[i][j])
      }
    }
    
    // 合并相同数字
    for (let i = 0; i < column.length - 1; i++) {
      if (column[i] === column[i + 1]) {
        column[i] *= 2
        score.value += column[i]
        column[i + 1] = 0
        
        if (column[i] === 2048 && !gameWon.value) {
          gameWon.value = true
        }
      }
    }
    
    const filteredColumn = column.filter(val => val !== 0)
    while (filteredColumn.length < 4) {
      filteredColumn.push(0)
    }
    
    for (let i = 0; i < 4; i++) {
      if (newGrid[i][j] !== filteredColumn[i]) {
        moved = true
      }
      newGrid[i][j] = filteredColumn[i]
    }
  }
  
  if (moved) {
    grid.value = newGrid
    addRandomTile()
    checkGameOver()
    updateBestScore()
  }
}

/**
 * 向下移动
 */
const moveDown = () => {
  let moved = false
  const newGrid = grid.value.map(row => [...row])
  
  for (let j = 0; j < 4; j++) {
    const column = []
    for (let i = 0; i < 4; i++) {
      if (newGrid[i][j] !== 0) {
        column.push(newGrid[i][j])
      }
    }
    
    // 从下到上合并
    for (let i = column.length - 1; i > 0; i--) {
      if (column[i] === column[i - 1]) {
        column[i] *= 2
        score.value += column[i]
        column[i - 1] = 0
        
        if (column[i] === 2048 && !gameWon.value) {
          gameWon.value = true
        }
      }
    }
    
    const filteredColumn = column.filter(val => val !== 0)
    while (filteredColumn.length < 4) {
      filteredColumn.unshift(0)
    }
    
    for (let i = 0; i < 4; i++) {
      if (newGrid[i][j] !== filteredColumn[i]) {
        moved = true
      }
      newGrid[i][j] = filteredColumn[i]
    }
  }
  
  if (moved) {
    grid.value = newGrid
    addRandomTile()
    checkGameOver()
    updateBestScore()
  }
}

/**
 * 检查游戏是否结束
 */
const checkGameOver = () => {
  // 检查是否还有空格
  for (let i = 0; i < 4; i++) {
    for (let j = 0; j < 4; j++) {
      if (grid.value[i][j] === 0) {
        return
      }
    }
  }
  
  // 检查是否还能合并
  for (let i = 0; i < 4; i++) {
    for (let j = 0; j < 4; j++) {
      const current = grid.value[i][j]
      if (
        (i < 3 && grid.value[i + 1][j] === current) ||
        (j < 3 && grid.value[i][j + 1] === current)
      ) {
        return
      }
    }
  }
  
  gameOver.value = true
}

/**
 * 更新最高分
 */
const updateBestScore = () => {
  if (score.value > bestScore.value) {
    bestScore.value = score.value
    if (process.client) {
      localStorage.setItem('2048-best-score', bestScore.value.toString())
    }
  }
}

/**
 * 处理键盘事件
 */
const handleKeyPress = (event) => {
  if (gameOver.value) return
  
  switch (event.key) {
    case 'ArrowLeft':
      event.preventDefault()
      moveLeft()
      break
    case 'ArrowRight':
      event.preventDefault()
      moveRight()
      break
    case 'ArrowUp':
      event.preventDefault()
      moveUp()
      break
    case 'ArrowDown':
      event.preventDefault()
      moveDown()
      break
  }
}

// 组件挂载时初始化
onMounted(() => {
  // 加载最高分
  if (process.client) {
    const savedBestScore = localStorage.getItem('2048-best-score')
    if (savedBestScore) {
      bestScore.value = parseInt(savedBestScore)
    }
  }
  
  // 开始新游戏
  newGame()
  
  // 添加键盘事件监听
  if (process.client) {
    window.addEventListener('keydown', handleKeyPress)
  }
})

// 组件卸载时清理
onUnmounted(() => {
  if (process.client) {
    window.removeEventListener('keydown', handleKeyPress)
  }
})
</script>

<style scoped>
.game-container {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
  text-align: center;
  background: #faf8ef;
  border-radius: 10px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
}

.game-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.game-header h1 {
  font-size: 48px;
  font-weight: bold;
  color: #776e65;
  margin: 0;
}

.score-container {
  display: flex;
  gap: 10px;
}

.score-box {
  background: #bbada0;
  padding: 10px 20px;
  border-radius: 6px;
  color: white;
  min-width: 80px;
}

.score-label {
  font-size: 12px;
  text-transform: uppercase;
  font-weight: bold;
  margin-bottom: 5px;
}

.score {
  font-size: 24px;
  font-weight: bold;
}

.game-intro {
  margin-bottom: 20px;
}

.game-intro p {
  color: #776e65;
  font-size: 16px;
  margin-bottom: 15px;
}

.restart-button, .continue-button {
  background: #8f7a66;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 6px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.3s;
  margin: 0 5px;
}

.restart-button:hover, .continue-button:hover {
  background: #9f8a76;
}

.grid-container {
  background: #bbada0;
  border-radius: 10px;
  padding: 10px;
  position: relative;
  margin: 0 auto;
  width: 340px;
  height: 340px;
}

.grid-row {
  display: flex;
  margin-bottom: 10px;
}

.grid-row:last-child {
  margin-bottom: 0;
}

.grid-cell {
  width: 70px;
  height: 70px;
  background: #cdc1b4;
  border-radius: 6px;
  margin-right: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32px;
  font-weight: bold;
  transition: all 0.15s ease-in-out;
  position: relative;
}

.grid-cell:last-child {
  margin-right: 0;
}

.grid-cell span {
  color: #776e65;
}

/* 不同数字的颜色样式 */
.tile-2 {
  background: #eee4da;
  color: #776e65;
}

.tile-4 {
  background: #ede0c8;
  color: #776e65;
}

.tile-8 {
  background: #f2b179;
  color: #f9f6f2;
}

.tile-16 {
  background: #f59563;
  color: #f9f6f2;
}

.tile-32 {
  background: #f67c5f;
  color: #f9f6f2;
}

.tile-64 {
  background: #f65e3b;
  color: #f9f6f2;
}

.tile-128 {
  background: #edcf72;
  color: #f9f6f2;
  font-size: 28px;
}

.tile-256 {
  background: #edcc61;
  color: #f9f6f2;
  font-size: 28px;
}

.tile-512 {
  background: #edc850;
  color: #f9f6f2;
  font-size: 28px;
}

.tile-1024 {
  background: #edc53f;
  color: #f9f6f2;
  font-size: 24px;
}

.tile-2048 {
  background: #edc22e;
  color: #f9f6f2;
  font-size: 24px;
  box-shadow: 0 0 30px 10px rgba(243, 215, 116, 0.4);
}

/* 更高数字的样式 */
.tile-4096, .tile-8192, .tile-16384, .tile-32768, .tile-65536 {
  background: #3c3a32;
  color: #f9f6f2;
  font-size: 20px;
}

.game-over-overlay, .game-won-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.9);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.game-over-message, .game-won-message {
  background: white;
  padding: 40px;
  border-radius: 10px;
  box-shadow: 0 0 30px rgba(0, 0, 0, 0.3);
  text-align: center;
}

.game-over-message h2, .game-won-message h2 {
  color: #776e65;
  font-size: 36px;
  margin-bottom: 20px;
}

.game-over-message p, .game-won-message p {
  color: #776e65;
  font-size: 18px;
  margin-bottom: 20px;
}
</style>
