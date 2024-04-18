<script setup lang="ts">
import { ref, onMounted, watch, onBeforeUnmount } from "vue";

const canvasRef = ref<HTMLCanvasElement | null>(null);
const paused = ref(false);
let cellSize = ref(25); // Initial cell size
let canvasSize = ref({ width: 0, height: 0 });
let numRows = ref(0);
let numCols = ref(0);
let initialCanvasHeight = 0; // Store the initial canvas height
let gridState: boolean[][] = [];
let isDragging = ref(false);

const handleMouseDown = (event: MouseEvent) => {
  isDragging.value = true;
  // handleMouseDrag(event);
};

const handleMouseUp = () => {
  isDragging.value = false;
};

const handleMouseMove = (event: MouseEvent) => {
  if (isDragging.value) {
    const canvas = canvasRef.value;
    if (!canvas) return;

    const rect = canvas.getBoundingClientRect();
    const scaleX = canvas.width / rect.width;
    const scaleY = canvas.height / rect.height;

    const x = (event.clientX - rect.left) * scaleX;
    const y = (event.clientY - rect.top) * scaleY;

    const col = Math.floor(x / cellSize.value);
    const row = Math.floor(y / cellSize.value);

    fillGridCell(col, row);

    // If you want to fill multiple cells while dragging, you can adjust the logic here
    // For example, you can keep track of the last filled cell and fill all cells in between

  }
};

const handleMouseLeave = () => {
  // End mouse drag event when mouse leaves the canvas
  isDragging.value = false;
};

onMounted(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  // Store the initial canvas height
  initialCanvasHeight = canvas.clientHeight;

  // Setup canvas context and initial game state
  setupCanvas(canvas);

  // Add window resize listener
  window.addEventListener('resize', handleWindowResize);
  canvas.addEventListener('mousedown', handleMouseDown);
  canvas.addEventListener('mouseup', handleMouseUp);
  canvas.addEventListener('mousemove', handleMouseMove);
  canvas.addEventListener('mouseleave', handleMouseLeave);
});

onBeforeUnmount(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  canvas.removeEventListener('mousedown', handleMouseDown);
  canvas.removeEventListener('mouseup', handleMouseUp);
  canvas.removeEventListener('mousemove', handleMouseMove);
  canvas.removeEventListener('mouseleave', handleMouseLeave);
});

const toggleAnimation = () => {
  paused.value = !paused.value;
  if (!paused.value) {
    // Start animation loop
  }
};

const fillGridCell = (col: number, row: number) => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  if (row < 0 || row >= numRows.value || col < 0 || col >= numCols.value) {
    // Row or column index is out of bounds
    return;
  }

  ctx.fillStyle = 'grey'; // Set desired fill color
  ctx.fillRect(col * cellSize.value, row * cellSize.value, cellSize.value, cellSize.value);
  gridState[row][col] = true;
};

const clearGridCell = (col: number, row: number) => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  // Clear the grid cell
  ctx.clearRect(col * cellSize.value, row * cellSize.value, cellSize.value, cellSize.value);

  // Update the grid state to indicate that this cell is now empty
  gridState[row][col] = false;
};

const clearAllFilledCells = () => {
  gridState.forEach((row, rowIndex) => {
    row.forEach((isFilled, colIndex) => {
      if (isFilled) {
        clearGridCell(colIndex, rowIndex);
      }
    });
  });
};

const handleCellSizeChange = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;
  const { width, height } = canvas.getBoundingClientRect();

  // Calculate new canvas dimensions that are multiples of the cell size
  const newWidth = Math.floor(width / cellSize.value) * cellSize.value;
  const newHeight = Math.floor(height / cellSize.value) * cellSize.value;

  // Update canvas dimensions
  canvas.width = newWidth;
  canvas.height = newHeight;
  drawGrid(ctx, canvas.width, canvas.height, cellSize.value);
  resetGridState();
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
  // Initialize gridState array with default values (e.g., all cells are initially empty)
  resetGridState();
  // Draw initial grid
  drawGrid(ctx, minDimension, minDimension, cellSize.value);
  // Initialize game state and draw initial grid
  // You can implement this part according to your game logic
};

const resetGridState = () => {
  if (!canvasRef.value) return;
  const newNumRows = Math.floor(canvasRef.value.height / cellSize.value);
  const newNumCols = Math.floor(canvasRef.value.width / cellSize.value);
  // console.log('newNumRows', newNumRows, 'newNumCols', newNumCols)
  // // Create a new gridState array with the updated dimensions
  const newGridState = Array.from({ length: newNumRows }, () =>
    Array.from({ length: newNumCols }, () => false)
  );
  // console.log('newGridState', newGridState)

  // // Copy existing values from old gridState to new gridState
  // for (let row = 0; row < Math.min(numRows.value, newNumRows); row++) {
  //   for (let col = 0; col < Math.min(numCols.value, newNumCols); col++) {
  //     try {
  //       newGridState[row][col] = gridState[row][col];
  //     }
  //     catch (e) {
  //       newGridState[row][col] = false;
  //     }
  //   }
  // }

  // // Update numRows, numCols, and gridState
  numRows.value = newNumRows;
  numCols.value = newNumCols;
  gridState = newGridState;
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
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, height);
    ctx.stroke();
  }

  // Draw horizontal grid lines
  for (let y = 0; y <= height; y += cellSize) {
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(width, y);
    ctx.stroke();
  }
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
      <div>Number of Rows: {{ gridState?.length }}</div>
      <div>Number of Columns: {{ gridState?.length }}</div>
      <div>Cell Size: {{ cellSize }}px</div>
      <input type='range' v-model.number="cellSize" min="10" max="30" step="1"></input>
    </div>
    <div class="controls">
      <button @click="clearAllFilledCells">Clear</button>
      <button @click="toggleAnimation">{{ paused ? "Resume" : "Pause" }}</button>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  width: 100%;
  height: 100%;
  max-width: 600px;
  /* Adjust maximum width as needed */
  margin: 0 auto;
  /* Center the container horizontally */
  position: relative;
}

canvas {
  width: 100%;
  height: auto;
  max-width: 100%;
  /* Ensure the canvas does not exceed the width of its container */
  display: block;
  /* Ensure the canvas behaves as a block-level element */
  border: 1px solid black;
  background-color: #222;
  /* Darker background color */
}

.controls {
  position: absolute;
  bottom: 10px;
  left: 10px;
  display: flex;
  gap: 10px;
  width: 100%;
}

button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  width: 50%;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

.debug-info div {
  margin-bottom: 5px;
}

input[type="range"] {
  width: 100%;
}
</style>
