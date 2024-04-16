<script setup lang="ts">
import { ref, onMounted, watch } from "vue";

const canvasRef = ref<HTMLCanvasElement | null>(null);
const paused = ref(false);
let cellSize = ref(25); // Initial cell size
let canvasSize = ref({ width: 0, height: 0 });
let numRows = ref(0);
let numCols = ref(0);

onMounted(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  // Setup canvas context and initial game state
  setupCanvas(canvas);

  // Add window resize listener
  window.addEventListener('resize', handleWindowResize);
});

const toggleAnimation = () => {
  paused.value = !paused.value;
  if (!paused.value) {
    // Start animation loop
  }
};

const setupCanvas = (canvas: HTMLCanvasElement) => {
  const ctx = canvas.getContext("2d");
  if (!ctx) return;

  const computedStyle = window.getComputedStyle(canvas);
  const width = parseInt(computedStyle.width);
  const height = parseInt(computedStyle.height);

  // Calculate number of rows and columns based on canvas size and cell size
  computeRowsAndCols(width, height);

  canvasSize.value = { width, height };

  // Resize canvas to fit exact number of cells
  canvas.width = numCols.value * cellSize.value;
  canvas.height = numRows.value * cellSize.value;
  ctx.strokeStyle = "rgb(255,255,255)";
  ctx.lineWidth = 1;
  // Draw initial grid
  drawGrid(ctx, width, height, cellSize.value);
  // Initialize game state and draw initial grid
  // You can implement this part according to your game logic
};

const computeRowsAndCols = (width: number, height: number) => {
  numRows.value = Math.floor(height / cellSize.value);
  numCols.value = Math.floor(width / cellSize.value);
};

const handleWindowResize = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const { width, height } = canvas.getBoundingClientRect();
  computeRowsAndCols(width, height);

  canvas.width = numCols.value * cellSize.value;
  canvas.height = numRows.value * cellSize.value;
  const ctx = canvas.getContext('2d');
  if (!ctx) return;
  // Redraw grid
  drawGrid(ctx, width, height, cellSize.value);
};

const drawGrid = (ctx: CanvasRenderingContext2D, width: number, height: number, cellSize: number) => {
  ctx.strokeStyle = "rgb(255,255,255)";
  ctx.lineWidth = 1;
  ctx.beginPath();
  for (let x = 0; x <= width; x += cellSize) {
    ctx.moveTo(x, 0);
    ctx.lineTo(x, height);
  }
  for (let y = 0; y <= height; y += cellSize) {
    ctx.moveTo(0, y);
    ctx.lineTo(width, y);
  }
  ctx.stroke();
};

watch(cellSize, (newCellSize, oldCellSize) => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const { width, height } = canvas.getBoundingClientRect();

  computeRowsAndCols(width, height);

  canvas.width = numCols.value * newCellSize;
  canvas.height = numRows.value * newCellSize;
});
</script>

<template>
  <div class="game-container">
    <canvas ref="canvasRef"></canvas>
    <div class="debug-info">
      <div>Canvas Size: {{ canvasSize.width }} x {{ canvasSize.height }}</div>
      <div>Number of Rows: {{ numRows }}</div>
      <div>Number of Columns: {{ numCols }}</div>
      <div>Cell Size: {{ cellSize }}px</div>
      <input type='range' v-model="cellSize" min="10" max="30"></input>
    </div>

    <button @click="toggleAnimation">{{ paused ? "Resume" : "Pause" }}</button>
  </div>
</template>

<style scoped>
.game-container {
  width: 100%;
  height: 100%;
  max-width: 600px; /* Adjust maximum width as needed */
  margin: 0 auto; /* Center the container horizontally */
  position: relative;
}

canvas {
  width: 100%;
  height: auto;
  max-width: 100%; /* Ensure the canvas does not exceed the width of its container */
  display: block; /* Ensure the canvas behaves as a block-level element */
  border: 1px solid black;
  background-color: black;
}

button {
  margin-top: 10px;
  width: 100%;
  position: absolute;
  bottom: 10px; /* Adjust button position as needed */
}
.debug-info div {
  margin-bottom: 5px;
}
input[type="range"] {
  width: 100%;
}
</style>
