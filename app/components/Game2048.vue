<template>
  <div class="game-container" :class="{ 'dark-theme': isDarkMode }">
    <div class="game-header">
      <h1>2048</h1>
      <div class="header-right">
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
        <button @click="toggleTheme" class="theme-toggle" :title="isDarkMode ? '切换到亮色主题' : '切换到暗色主题'">
          <span v-if="isDarkMode">🌙</span>
          <span v-else>☀️</span>
        </button>
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
          :data-row="rowIndex"
          :data-col="colIndex"
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
const animatingCells = ref([]) // 用于存储动画中的方块

// 触摸手势相关状态
const touchStartX = ref(0)
const touchStartY = ref(0)
const touchStartTime = ref(0)

// 主题相关状态
const isDarkMode = ref(false)

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
    
    // 添加新方块出现动画
    const cellElement = document.querySelector(`[data-row="${randomCell.row}"][data-col="${randomCell.col}"]`)
    if (cellElement) {
      cellElement.classList.add('tile-new')
      setTimeout(() => {
        cellElement.classList.remove('tile-new')
      }, 300)
    }
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
 * 切换主题
 */
const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
  if (process.client) {
    localStorage.setItem('2048-dark-mode', isDarkMode.value.toString())
  }
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
  const mergedCells = []
  
  for (let i = 0; i < 4; i++) {
    const row = newGrid[i].filter(val => val !== 0)
    
    // 合并相同的数字
    for (let j = 0; j < row.length - 1; j++) {
      if (row[j] === row[j + 1]) {
        row[j] *= 2
        score.value += row[j]
        row[j + 1] = 0
        mergedCells.push({ row: i, col: j })
        
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
    
    // 添加合并动画
    setTimeout(() => {
      mergedCells.forEach(cell => {
        const cellElement = document.querySelector(`[data-row="${cell.row}"][data-col="${cell.col}"]`)
        if (cellElement) {
          cellElement.classList.add('tile-merged')
          setTimeout(() => {
            cellElement.classList.remove('tile-merged')
          }, 300)
        }
      })
    }, 50)
    
    setTimeout(() => {
      addRandomTile()
      checkGameOver()
      updateBestScore()
    }, 150)
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

/**
 * 处理触摸开始
 */
const handleTouchStart = (event) => {
  if (gameOver.value) return
  
  const touch = event.touches[0]
  touchStartX.value = touch.clientX
  touchStartY.value = touch.clientY
  touchStartTime.value = Date.now()
}

/**
 * 处理触摸移动（阻止页面滚动）
 */
const handleTouchMove = (event) => {
  event.preventDefault()
}

/**
 * 处理触摸结束
 */
const handleTouchEnd = (event) => {
  if (gameOver.value) return
  
  const touch = event.changedTouches[0]
  const deltaX = touch.clientX - touchStartX.value
  const deltaY = touch.clientY - touchStartY.value
  const deltaTime = Date.now() - touchStartTime.value
  
  // 最小滑动距离和最大时间限制
  const minSwipeDistance = 50
  const maxSwipeTime = 1000
  
  if (deltaTime > maxSwipeTime) return
  
  const absDeltaX = Math.abs(deltaX)
  const absDeltaY = Math.abs(deltaY)
  
  if (Math.max(absDeltaX, absDeltaY) < minSwipeDistance) return
  
  if (absDeltaX > absDeltaY) {
    // 水平滑动
    if (deltaX > 0) {
      moveRight()
    } else {
      moveLeft()
    }
  } else {
    // 垂直滑动
    if (deltaY > 0) {
      moveDown()
    } else {
      moveUp()
    }
  }
}

// 组件挂载时初始化
onMounted(() => {
  // 加载最高分和主题设置
  if (process.client) {
    const savedBestScore = localStorage.getItem('2048-best-score')
    if (savedBestScore) {
      bestScore.value = parseInt(savedBestScore)
    }
    
    const savedDarkMode = localStorage.getItem('2048-dark-mode')
    if (savedDarkMode) {
      isDarkMode.value = savedDarkMode === 'true'
    }
  }
  
  // 开始新游戏
  newGame()
  
  // 添加键盘事件监听
  if (process.client) {
    window.addEventListener('keydown', handleKeyPress)
    
    // 添加触摸事件监听
    const gameContainer = document.querySelector('.game-container')
    if (gameContainer) {
      gameContainer.addEventListener('touchstart', handleTouchStart, { passive: false })
      gameContainer.addEventListener('touchmove', handleTouchMove, { passive: false })
      gameContainer.addEventListener('touchend', handleTouchEnd, { passive: false })
    }
  }
})

// 组件卸载时清理
onUnmounted(() => {
  if (process.client) {
    window.removeEventListener('keydown', handleKeyPress)
    
    // 移除触摸事件监听
    const gameContainer = document.querySelector('.game-container')
    if (gameContainer) {
      gameContainer.removeEventListener('touchstart', handleTouchStart)
      gameContainer.removeEventListener('touchmove', handleTouchMove)
      gameContainer.removeEventListener('touchend', handleTouchEnd)
    }
  }
})
</script>

<style scoped>
.game-container {
  max-width: 500px;
  margin: 0 auto;
  padding: 30px;
  font-family: 'Arial', sans-serif;
  text-align: center;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3), 0 0 0 1px rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
  user-select: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  touch-action: manipulation;
}

/* 移动设备适配 */
@media (max-width: 768px) {
  .game-container {
    max-width: 350px;
    padding: 20px;
    margin: 10px auto;
  }
}

@media (max-width: 480px) {
  .game-container {
    max-width: 320px;
    padding: 15px;
    margin: 5px auto;
  }
}

.game-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 15px;
}

@media (max-width: 480px) {
  .game-header {
    flex-direction: column;
    gap: 15px;
    margin-bottom: 15px;
  }
  
  .header-right {
    flex-direction: column;
    gap: 10px;
  }
}

.game-header h1 {
  font-size: 56px;
  font-weight: bold;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin: 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

@media (max-width: 480px) {
  .game-header h1 {
    font-size: 42px;
  }
}

.score-container {
  display: flex;
  gap: 10px;
}

.theme-toggle {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  cursor: pointer;
  font-size: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
  transition: all 0.3s ease;
}

.theme-toggle:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.4);
}

.theme-toggle:active {
  transform: translateY(0);
}

.score-box {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 15px 25px;
  border-radius: 12px;
  color: white;
  min-width: 80px;
  box-shadow: 0 8px 16px rgba(102, 126, 234, 0.3);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.score-box:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 24px rgba(102, 126, 234, 0.4);
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
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 12px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  margin: 0 8px;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

.restart-button:hover, .continue-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.4);
}

.restart-button:active, .continue-button:active {
  transform: translateY(0);
}

.grid-container {
  background: rgba(187, 173, 160, 0.8);
  backdrop-filter: blur(5px);
  border-radius: 16px;
  padding: 15px;
  position: relative;
  margin: 0 auto;
  width: 340px;
  height: 340px;
  box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.1);
}

@media (max-width: 768px) {
  .grid-container {
    width: 280px;
    height: 280px;
    padding: 12px;
  }
}

@media (max-width: 480px) {
  .grid-container {
    width: 260px;
    height: 260px;
    padding: 10px;
  }
}

.grid-row {
  display: flex;
  margin-bottom: 12px;
}

.grid-row:last-child {
  margin-bottom: 0;
}

.grid-cell {
  width: 70px;
  height: 70px;
  background: rgba(238, 228, 218, 0.35);
  border-radius: 8px;
  margin-right: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32px;
  font-weight: bold;
  transition: all 0.15s ease-in-out;
  position: relative;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.1);
}

@media (max-width: 768px) {
  .grid-cell {
    width: 58px;
    height: 58px;
    margin-right: 10px;
    font-size: 28px;
  }
}

@media (max-width: 480px) {
  .grid-cell {
    width: 54px;
    height: 54px;
    margin-right: 8px;
    font-size: 24px;
  }
}

.grid-cell:last-child {
  margin-right: 0;
}

.grid-cell span {
  color: #776e65;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

/* 新方块出现动画 */
.tile-new {
  animation: tileAppear 0.3s ease-out;
}

@keyframes tileAppear {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  50% {
    transform: scale(1.1);
    opacity: 0.8;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

/* 方块合并动画 */
.tile-merged {
  animation: tileMerge 0.3s ease-out;
}

@keyframes tileMerge {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
  }
}

/* 不同数字的颜色样式 - 现代渐变版本 */
.tile-2 {
  background: linear-gradient(135deg, #eee4da 0%, #f2f0e8 100%);
  color: #776e65;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.tile-4 {
  background: linear-gradient(135deg, #ede0c8 0%, #f5e9d3 100%);
  color: #776e65;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.tile-8 {
  background: linear-gradient(135deg, #f2b179 0%, #f5c394 100%);
  color: #f9f6f2;
  box-shadow: 0 4px 12px rgba(242, 177, 121, 0.3);
}

.tile-16 {
  background: linear-gradient(135deg, #f59563 0%, #f7a67e 100%);
  color: #f9f6f2;
  box-shadow: 0 4px 12px rgba(245, 149, 99, 0.3);
}

.tile-32 {
  background: linear-gradient(135deg, #f67c5f 0%, #f88d7a 100%);
  color: #f9f6f2;
  box-shadow: 0 6px 16px rgba(246, 124, 95, 0.3);
}

.tile-64 {
  background: linear-gradient(135deg, #f65e3b 0%, #f87056 100%);
  color: #f9f6f2;
  box-shadow: 0 6px 16px rgba(246, 94, 59, 0.3);
}

.tile-128 {
  background: linear-gradient(135deg, #edcf72 0%, #f0d687 100%);
  color: #f9f6f2;
  font-size: 28px;
  box-shadow: 0 8px 20px rgba(237, 207, 114, 0.4);
}

@media (max-width: 768px) {
  .tile-128 {
    font-size: 24px;
  }
}

@media (max-width: 480px) {
  .tile-128 {
    font-size: 20px;
  }
}

.tile-256 {
  background: linear-gradient(135deg, #edcc61 0%, #f0d376 100%);
  color: #f9f6f2;
  font-size: 28px;
  box-shadow: 0 8px 20px rgba(237, 204, 97, 0.4);
}

@media (max-width: 768px) {
  .tile-256 {
    font-size: 24px;
  }
}

@media (max-width: 480px) {
  .tile-256 {
    font-size: 20px;
  }
}

.tile-512 {
  background: linear-gradient(135deg, #edc850 0%, #f0cf65 100%);
  color: #f9f6f2;
  font-size: 28px;
  box-shadow: 0 8px 20px rgba(237, 200, 80, 0.4);
}

@media (max-width: 768px) {
  .tile-512 {
    font-size: 24px;
  }
}

@media (max-width: 480px) {
  .tile-512 {
    font-size: 20px;
  }
}

.tile-1024 {
  background: linear-gradient(135deg, #edc53f 0%, #f0cc54 100%);
  color: #f9f6f2;
  font-size: 24px;
  box-shadow: 0 10px 24px rgba(237, 197, 63, 0.5);
}

@media (max-width: 768px) {
  .tile-1024 {
    font-size: 20px;
  }
}

@media (max-width: 480px) {
  .tile-1024 {
    font-size: 18px;
  }
}

.tile-2048 {
  background: linear-gradient(135deg, #edc22e 0%, #f0c943 100%);
  color: #f9f6f2;
  font-size: 24px;
  box-shadow: 0 12px 30px rgba(237, 194, 46, 0.6), 0 0 30px 10px rgba(243, 215, 116, 0.4);
  animation: pulse 2s infinite;
}

@media (max-width: 768px) {
  .tile-2048 {
    font-size: 20px;
  }
}

@media (max-width: 480px) {
  .tile-2048 {
    font-size: 18px;
  }
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
    box-shadow: 0 12px 30px rgba(237, 194, 46, 0.6), 0 0 30px 10px rgba(243, 215, 116, 0.4);
  }
  50% {
    transform: scale(1.02);
    box-shadow: 0 15px 36px rgba(237, 194, 46, 0.8), 0 0 40px 15px rgba(243, 215, 116, 0.6);
  }
}

/* 更高数字的样式 */
.tile-4096, .tile-8192, .tile-16384, .tile-32768, .tile-65536 {
  background: linear-gradient(135deg, #3c3a32 0%, #4a4847 100%);
  color: #f9f6f2;
  font-size: 20px;
  box-shadow: 0 12px 30px rgba(60, 58, 50, 0.5);
}

.game-over-overlay, .game-won-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.game-over-message, .game-won-message {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  padding: 40px;
  border-radius: 20px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  text-align: center;
  transform: scale(0.9);
  animation: modalAppear 0.3s ease-out forwards;
}

@keyframes modalAppear {
  to {
    transform: scale(1);
  }
}

.game-over-message h2, .game-won-message h2 {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-size: 36px;
  margin-bottom: 20px;
}

.game-over-message p, .game-won-message p {
  color: #776e65;
  font-size: 18px;
  margin-bottom: 20px;
}

/* 暗色主题样式 */
.game-container.dark-theme {
  background: rgba(45, 45, 45, 0.95);
  color: #e8e8e8;
}

.game-container.dark-theme .game-intro p {
  color: #cccccc;
}

.game-container.dark-theme .grid-container {
  background: rgba(60, 60, 60, 0.8);
}

.game-container.dark-theme .grid-cell {
  background: rgba(80, 80, 80, 0.5);
}

.game-container.dark-theme .grid-cell span {
  color: #e8e8e8;
}

.game-container.dark-theme .game-over-message,
.game-container.dark-theme .game-won-message {
  background: rgba(45, 45, 45, 0.95);
  color: #e8e8e8;
}

.game-container.dark-theme .game-over-message h2,
.game-container.dark-theme .game-won-message h2 {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.game-container.dark-theme .game-over-message p,
.game-container.dark-theme .game-won-message p {
  color: #cccccc;
}
</style>
