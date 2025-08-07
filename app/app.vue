<template>
  <div class="app">
    <NuxtRouteAnnouncer />
    <Game2048 />
  </div>
</template>

<style>
body {
  margin: 0;
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  background-attachment: fixed;
  font-family: 'Arial', sans-serif;
  min-height: 100vh;
  transition: background 0.3s ease;
}

/* 暗色主题背景 */
body:has(.dark-theme) {
  background: linear-gradient(135deg, #2c3e50 0%, #34495e 100%);
}

.app {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
