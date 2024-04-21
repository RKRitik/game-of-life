<script setup lang="ts">
import { ref, onMounted, watch, onBeforeUnmount } from "vue";

const canvasRef = ref<HTMLCanvasElement | null>(null);
const paused = ref(false);
let cellSize = ref(10); // Initial cell size
let canvasSize = ref({ width: 0, height: 0 });
let numRows = ref(0);
let numCols = ref(0);
let initialCanvasHeight = 0; // Store the initial canvas height
let gridState: boolean[][] = [];
let isDragging = ref(false);
const targetFrameRate = 5; // Target frame rate in frames per second
const frameDelay = 1000 / targetFrameRate; // Delay in milliseconds between frames


const handleMouseDown = (_event: MouseEvent) => {
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
  }
};

const handleMouseLeave = () => {
  // End mouse drag event when mouse leaves the canvas
  isDragging.value = false;
};

const handleCanvasClick = (event: MouseEvent) => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / rect.width;
  const scaleY = canvas.height / rect.height;

  const x = (event.clientX - rect.left) * scaleX;
  const y = (event.clientY - rect.top) * scaleY;

  const col = Math.floor(x / cellSize.value);
  const row = Math.floor(y / cellSize.value);

  fillGridCell(col, row);
};

const lifeSpawner = () => {
  let choice = Math.floor(Math.random() * 4);
  switch (choice) {
    case 0: //1. static 4x4
      spawnStaticPattern(); break;
    case 1:// 2. blinker
      spawnBlinker();
      break;
    case 2: // 3. glider
      spawnGlider();
      break;
    case 3:// 4. pulsar
      spawnPulsar();
      break;
  }
}

const spawnStaticPattern = () => {
  const pattern = [
    [true, true, false, false],
    [true, true, false, false],
    [false, false, true, true],
    [false, false, true, true]
  ];

  updateGridState(pattern);
};

const spawnBlinker = () => {
  const pattern = [
    [false, true, false],
    [false, true, false],
    [false, true, false]
  ];

  updateGridState(pattern);
};

const spawnGlider = () => {
  const pattern = [
    [false, true, false],
    [false, false, true],
    [true, true, true]
  ];

  updateGridState(pattern);
};

const spawnPulsar = () => {
  const pattern = [
    [false, true, true, true, false, false, true, true, true, false],
    [true, false, false, false, true, true, false, false, false, true],
    [true, false, false, false, true, true, false, false, false, true],
    [true, false, false, false, true, true, false, false, false, true],
    [false, true, true, true, false, false, true, true, true, false],
    [false, true, true, true, false, false, true, true, true, false],
    [true, false, false, false, true, true, false, false, false, true],
    [true, false, false, false, true, true, false, false, false, true],
    [true, false, false, false, true, true, false, false, false, true],
    [false, true, true, true, false, false, true, true, true, false]
  ];

  updateGridState(pattern);
};

const updateGridState = (pattern: boolean[][]) => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  // Get the maximum start row and column values to ensure the pattern fits within bounds
  const maxStartRow = numRows.value - pattern.length;
  const maxStartCol = numCols.value - pattern[0].length;

  // Generate random start row and column values within bounds
  const startRow = Math.floor(Math.random() * (maxStartRow + 1));
  const startCol = Math.floor(Math.random() * (maxStartCol + 1));


  // Update the grid state with the pattern
  for (let i = 0; i < pattern.length; i++) {
    for (let j = 0; j < pattern[i].length; j++) {
      gridState[startRow + i][startCol + j] = pattern[i][j];
    }
  }

  // Redraw the grid on the canvas
  redrawGrid(canvas.width, canvas.height, cellSize.value, gridState);
};

const randomLives = () => {
  if (paused.value) return;
  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  // Create a new generation array to store the next state
  const nextGeneration = [];

  // Iterate through each cell in the grid
  for (let row = 0; row < numRows.value; row++) {
    const newRow = [];
    for (let col = 0; col < numCols.value; col++) {
      // Randomly determine whether the cell should be alive or dead
      const isAlive = Math.random() < 0.5; // 50% chance of being alive
      newRow.push(isAlive);
    }
    nextGeneration.push(newRow);
  }

  // Update the grid state with the next generation
  gridState = nextGeneration;

  // Redraw the entire grid on the canvas using the nextGeneration array
  redrawGrid(canvas.width, canvas.height, cellSize.value, nextGeneration);
}
onMounted(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  // Store the initial canvas height
  initialCanvasHeight = canvas.clientHeight;

  // Setup canvas context and initial game state
  setupCanvas(canvas);
  randomLives();

  // Add window resize listener
  window.addEventListener('resize', handleWindowResize);
  canvas.addEventListener('mousedown', handleMouseDown);
  canvas.addEventListener('mouseup', handleMouseUp);
  canvas.addEventListener('mousemove', handleMouseMove);
  canvas.addEventListener('mouseleave', handleMouseLeave);
  canvas.addEventListener('click', handleCanvasClick)
  computeNextGeneration();
});

onBeforeUnmount(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  canvas.removeEventListener('mousedown', handleMouseDown);
  canvas.removeEventListener('mouseup', handleMouseUp);
  canvas.removeEventListener('mousemove', handleMouseMove);
  canvas.removeEventListener('mouseleave', handleMouseLeave);
  canvas.removeEventListener('click', handleCanvasClick);
});

const toggleAnimation = () => {
  paused.value = !paused.value;
  if (!paused.value) {
    computeNextGeneration();
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
  // ctx.clearRect(0, 0, width, height); // Clear the canvas
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

const getAliveNeighbours = (col: number, row: number) => {
  let aliveNeighbours = 0;

  // Define the offsets for neighboring cells
  const offsets = [
    [-1, -1], [-1, 0], [-1, 1],
    [0, -1], /* Current cell */[0, 1],
    [1, -1], [1, 0], [1, 1]
  ];

  // Iterate through each neighboring cell
  for (const [dx, dy] of offsets) {
    const newRow = row + dx;
    const newCol = col + dy;
    try {
      // Check if the neighboring cell is within the grid bounds
      if (newRow >= 0 && newRow < numRows.value && newCol >= 0 && newCol < numCols.value) {
        // If the neighboring cell is alive, increment the count
        if (gridState[newRow][newCol]) {
          aliveNeighbours++;
        }
      }
    } catch (e) { }
  }

  return aliveNeighbours;
};

const computeNextGeneration = () => {
  if (paused.value) return;
  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;
  const nextGeneration = Array.from({ length: numRows.value }, () =>
    Array.from({ length: numCols.value }, () => false)
  );

  // Iterate through each cell in the grid
  for (let row = 0; row < numRows.value; row++) {
    let valRow = gridState[row];
    if (!valRow) continue;
    for (let col = 0; col < numCols.value; col++) {
      // Count the number of alive neighbors for the current cell
      const aliveNeighbours = getAliveNeighbours(col, row);

      // Apply the rules of the Game of Life to determine the state of the cell in the next generation
      if (gridState[row][col]) {
        // Any live cell with fewer than two live neighbors dies
        // Any live cell with two or three live neighbors lives on to the next generation
        // Any live cell with more than three live neighbors dies
        nextGeneration[row][col] = aliveNeighbours === 2 || aliveNeighbours === 3;
      } else {
        // Any dead cell with exactly three live neighbors becomes a live cell
        nextGeneration[row][col] = aliveNeighbours === 3;
      }
    }
  }

  // Update the gridState with the next generation
  gridState = nextGeneration;

  // Redraw the entire grid on the canvas using the nextGeneration array
  redrawGrid(canvas.width, canvas.height, cellSize.value, nextGeneration);
  setTimeout(() => {
    requestAnimationFrame(computeNextGeneration);
  }, frameDelay);
}

const redrawGrid = (width: number, height: number, cellSize: number, nextGeneration: boolean[][]) => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  // Clear the canvas
  ctx.clearRect(0, 0, width, height);

  // Iterate through the nextGeneration array and draw each cell
  for (let row = 0; row < numRows.value; row++) {
    for (let col = 0; col < numCols.value; col++) {
      if (nextGeneration[row][col]) {
        ctx.fillStyle = 'grey';
        ctx.fillRect(col * cellSize, row * cellSize, cellSize, cellSize);
      }
    }
  }
};

watch(cellSize, (_newCellSize, _oldCellSize) => {
  handleCellSizeChange();
});
</script>

<script lang="ts">
import GameControls from './GameControls.vue';
export default {
  props: {
    isFull: {
      type: Boolean,
      required: true
    }
  },
};
</script>

<template>
  <GameControls :paused="paused" ref="controls" @onClear="clearAllFilledCells" @onToggle="toggleAnimation"
    @onRandom="randomLives" @onSpawnLife="lifeSpawner" />
  <div :class="{ 'game-container-sq': !isFull, 'game-container': true }">
    <canvas ref="canvasRef"></canvas>
    <div class="debug-info">
      <div>Canvas Size: {{ canvasSize.width }} x {{ canvasSize.height }}</div>
      <div>Number of Rows: {{ gridState?.length }}</div>
      <div>Number of Columns: {{ gridState?.length }}</div>
      <div>Cell Size: {{ cellSize }}px</div>
      <input type='range' v-model.number="cellSize" min="10" max="60" step="5"></input>
    </div>
    <!-- <div class="controls">
      <button @click="clearAllFilledCells">Clear</button>
      <button @click="toggleAnimation">{{ paused ? "Resume" : "Pause" }}</button>
      <button @click="randomLives">random</button>
      <button @click="lifeSpawner">life Spawner</button>
    </div> -->
  </div>
</template>

<style scoped>
.game-container {
  width: 100%;
  height: 100%;
  margin: 0 auto;
  position: relative;
}

.game-container-sq {
  max-width: 600px;
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
  display: none;
}

input[type="range"] {
  width: 100%;
  display: none;
}
</style>
