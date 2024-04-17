<script setup lang="ts">
import { ref, onMounted, watch } from "vue";

const canvasRef = ref<HTMLCanvasElement | null>(null);
const paused = ref(false);
let cellSize = ref(25); // Initial cell size
let canvasSize = ref({ width: 0, height: 0 });
let numRows = ref(0);
let numCols = ref(0);
let initialCanvasHeight = 0; // Store the initial canvas height

onMounted(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  // Store the initial canvas height
  initialCanvasHeight = canvas.clientHeight;

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

const handleCellSizeChange = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;
  // Recalculate number of rows and columns based on new cell size
  computeRowsAndCols();

  // Calculate new canvas width and height based on the number of rows and columns
  // const newWidth = numCols.value * cellSize.value;
  // const newHeight = numRows.value * cellSize.value;

  // // Adjust canvas size
  // canvas.width = newWidth;
  // canvas.height = newHeight;

  // Redraw grid lines
  console.log('typeof ', typeof cellSize.value)
  drawGrid(ctx,  canvas.width, canvas.height, cellSize.value);
};

const setupCanvas = (canvas: HTMLCanvasElement) => {
  const ctx = canvas.getContext("2d");
  if (!ctx) return;

  // Calculate number of rows and columns based on canvas size and cell size
  computeRowsAndCols();
// Ensure canvas size is square
  const minDimension = Math.min(numRows.value, numCols.value) * cellSize.value;
  canvas.width = minDimension;
  canvas.height = minDimension;

  canvasSize.value = { width: minDimension, height: minDimension };
  // Draw initial grid
  drawGrid(ctx, minDimension, minDimension, cellSize.value);
  // Initialize game state and draw initial grid
  // You can implement this part according to your game logic
};

const computeRowsAndCols = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  numRows.value = Math.floor(canvas.clientHeight / cellSize.value);
  numCols.value = Math.floor(canvas.clientWidth / cellSize.value);
};

const handleWindowResize = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  computeRowsAndCols();

  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  // Redraw grid
  drawGrid(ctx, canvas.clientWidth, initialCanvasHeight, cellSize.value);
};

const drawGrid = (ctx: CanvasRenderingContext2D, width: number, height: number, cellSize: number) => {
  ctx.clearRect(0, 0, width, height); // Clear the canvas
  ctx.strokeStyle = "rgb(255,255,255)";
  ctx.lineWidth = 1;
  console.log('cellSize', cellSize)
  // Draw vertical grid lines
  for (let x = 0; x <= width; x += cellSize) {
    console.log('vertical line :', x)
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, height);
    ctx.stroke();
  }

  // Draw horizontal grid lines
  for (let y = 0; y <= height; y += cellSize) {
    console.log('horizontal line :', y)
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(width, y);
    ctx.stroke();
  }
};

const computeMaxCellSize = () => {
  const canvas = canvasRef.value;
  if (!canvas) return 0;

  // Calculate maximum allowed cell size based on canvas dimensions and number of rows/columns
  const maxWidthCellSize = Math.floor(canvas.width / numCols.value);
  const maxHeightCellSize = Math.floor(canvas.height / numRows.value);

  // Choose the smaller value as the maximum allowed cell size
  return Math.min(maxWidthCellSize, maxHeightCellSize);
};

watch(cellSize, (newCellSize, oldCellSize) => {
  handleCellSizeChange();
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
      <input type='range' v-model.number="cellSize" min="10" max="30" step="1"></input>
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
  background-color: #222; /* Darker background color */
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
