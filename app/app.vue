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
  background: #faf8ef;
  font-family: 'Arial', sans-serif;
}

.app {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
