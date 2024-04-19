<script setup lang="ts">
import GameOfLife from "./components/GameOfLife.vue";
</script>
<template>
  <div>
    <header class="app-header">
      <h1>Game of Life</h1>
      <span>fullscreen</span>
      <div class="header-links">
        <a href="/about">About the Game</a>
        <a href="https://github.com/RKRitik/game-of-life">GitHub</a>
        <a href="https://www.linkedin.com/in/ritik-khatri-673b62115/">Linkedin</a>
      </div>
    </header>

    <div class="canvas-container">
      <GameOfLife />
    </div>
  </div>
</template>

<style scoped>
.app-header {
  background-color: #333;
  color: white;
  padding: 10px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.app-header h1 {
  margin: 0;
}

.header-links a {
  color: white;
  text-decoration: none;
  margin-right: 10px;
}
</style>
