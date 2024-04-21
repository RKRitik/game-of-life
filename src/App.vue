<script setup lang="ts">
import { ref } from "vue";

import GameOfLife from "./components/GameOfLife.vue";
let full = ref(false);
</script>
<template>
  <div class="app">
    <header class="app-header">
      <h1>Game of Life</h1>
      <span class="full-cta" @click="full = !full">Fullscreen</span>
      <div class="header-links">
        <a href="/about">About the Game</a>
        <a href="https://github.com/RKRitik/game-of-life">GitHub</a>
        <a href="https://www.linkedin.com/in/ritik-khatri-673b62115/">Linkedin</a>
      </div>
    </header>

    <div class="canvas-container">
      <GameOfLife :isFull="full" />
    </div>
  </div>
</template>

<style scoped>
.full-cta {
  cursor: pointer;
}

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

.app,
.canvas-container {
  width: 100%;
  height: 100%;
}
</style>
