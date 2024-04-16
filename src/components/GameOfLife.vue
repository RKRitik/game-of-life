/** At each step in time, the following transitions occur: Any live cell with
fewer than two live neighbors dies, as if by underpopulation. Any live cell with
two or three live neighbors lives on to the next generation. Any live cell with
more than three live neighbors dies, as if by overpopulation. Any dead cell with
exactly three live neighbors becomes a live cell, as if by reproduction. */
<script setup lang="ts">
import { onMounted, ref, onBeforeUnmount } from "vue";
const canvasRef = ref<HTMLCanvasElement | null>(null);
const canvasDimensions = ref<{ height: number; width: number } | null>();
const canvasLives = ref<Array<Array<boolean>>>();
const paused = ref(false);
const canvasRows = ref(0),
  canvasCols = ref(0);
const tileHeight = 25;

const toggleAnimation = () => {
  let newValue = !paused.value;
  paused.value = newValue;
  if (!newValue) updateLives();
};

var drawGrid = function (ctx: any, w: number, h: number, step: number) {
  ctx.beginPath();
  for (var x = 0; x <= w; x += step) {
    ctx.moveTo(x, 0);
    ctx.lineTo(x, h);
  }
  // set the color of the line
  ctx.strokeStyle = "rgb(255,255,255)";
  ctx.lineWidth = 1;
  // the stroke will actually paint the current path
  ctx.stroke();
  // for the sake of the example 2nd path
  ctx.beginPath();
  for (var y = 0; y <= h; y += step) {
    ctx.moveTo(0, y);
    ctx.lineTo(w, y);
  }
  // set the color of the line
  ctx.strokeStyle = "rgb(255,255,255)";
  // just for fun
  // ctx.lineWidth = 5;
  // for your original question - you need to stroke only once
  ctx.stroke();
};

const flowLife = () => {
  if (!canvasDimensions.value) return;
  // console.log("canvasLives.value", canvasLives.value);
  let xRand = Math.random() * canvasDimensions.value.width,
    yRand = Math.random() * canvasDimensions.value.height;
  // console.log("xRand", xRand);
  //1% chance to grow this life.
  let surviveChance = Math.random();
  // if (surviveChance > 0.99) addLife({ x: xRand, y: yRand });
  requestAnimationFrame(flowLife);
};

const addLife = (val: { x: number; y: number }) => {
  console.log("add life at ", val);
  const x = Math.floor(val.x),
    y = Math.floor(val.y);
  let colNum = Math.floor(x / canvasCols.value),
    rowNum = Math.floor(y / canvasRows.value);
  drawLife(colNum * tileHeight, rowNum * tileHeight);
  console.log("canvasLives", canvasLives.value);
  try {
    if (canvasLives?.value) canvasLives.value[colNum][rowNum] = true;
  } catch (e) {
    console.warn("error at", val);
  }
  if (!paused.value) updateLives();
  //loop through all lives and check possiblities
  //1. check for current cells 8 neighbours.
};

const updateLives = () => {
  if (!canvasLives.value) return;
  for (let i = 0; i < canvasCols.value; i++) {
    for (let j = 0; j < canvasRows.value; j++) {
      let alive = getAliveNeighbours(i, j);
      if (alive < 2 || alive > 3) {
        //this life dies
        if (canvasLives.value[i]) canvasLives.value[i][j] = false;
        drawLife(i * tileHeight, j * tileHeight, false);
      } else if (alive === 3) {
        if (canvasLives.value[i]) canvasLives.value[i][j] = true;
        drawLife(i * tileHeight, j * tileHeight);
      }
    }
  }
};

const getAliveNeighbours = (x: number, y: number) => {
  if (!canvasLives?.value) return 0;
  let num = 0;
  for (let i = x - 1; i <= x + 1; i++) {
    for (let j = y - 1; j <= y + 1; j++) {
      // Skip the current cell
      if (i === x && j === y) continue;

      // Check if the neighbor is within the bounds of the grid
      if (
        i >= 0 &&
        i < canvasLives.value.length &&
        j >= 0 &&
        j < canvasLives.value[0].length
      ) {
        if (canvasLives.value[i][j] === true) num++;
      }
    }
  }
  return num;
};

const drawLife = (x: number, y: number, alive = true) => {
  const ctx = (canvasRef.value as HTMLCanvasElement)?.getContext("2d");
  if (!ctx) return;
  const canvas = canvasRef.value;
  if (!canvas) return;
  ctx.fillStyle = "white";
  if (!alive) ctx.fillStyle = "black";

  ctx.fillRect(x, y, tileHeight, tileHeight);
  // toggleAnimation();
  drawGrid(ctx, canvas.width, canvas.height, tileHeight);
};

const setupCanvas = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  const ctx = (canvasRef.value as HTMLCanvasElement)?.getContext("2d");

  const resizeCanvas = () => {
    //redraw also?
    // canvas.width = window.innerWidth;
    // canvas.height = window.innerHeight;
    canvasDimensions.value = { width: canvas.width, height: canvas.height };
    window.innerHeight;
    canvasCols.value = Math.floor(canvasDimensions.value.width / tileHeight);
    canvasRows.value = Math.floor(canvasDimensions.value.height / tileHeight);
    canvasLives.value = Array.from({ length: canvasRows.value }, () =>
      Array.from({ length: canvasCols.value }, () => false)
    );
    drawGrid(ctx, canvas.width, canvas.height, tileHeight);
  };

  const handleClick = (val: any) => {
    addLife({ x: val.clientX, y: val.clientY });
  };

  resizeCanvas();
  window.addEventListener("resize", resizeCanvas);
  canvas.addEventListener("click", handleClick);
  flowLife();

  onBeforeUnmount(() => {
    window.removeEventListener("resize", resizeCanvas);
    canvasRef.value?.removeEventListener("click", handleClick);
    paused.value = true; // Stop animation loop
  });
};

onMounted(setupCanvas);
const pausedButtonText = () => {
  return paused.value ? "resume" : "pause";
};

const addPattern = () => {
  drawLife(0 * tileHeight, 5 * tileHeight);
  drawLife(1 * tileHeight, 5 * tileHeight);
  drawLife(2 * tileHeight, 5 * tileHeight);
  if (!canvasLives?.value) return;
  canvasLives.value[4][3] = true;
  canvasLives.value[4][4] = true;
  canvasLives.value[4][5] = true;
  updateLives();
};
</script>

<template>
  <canvas ref="canvasRef" id="grid"></canvas>
  <button class="toggle" @click="toggleAnimation">
    {{ pausedButtonText() }}
  </button>
  <div>
    canvasDimensions : height : {{ canvasDimensions?.height }} ; width :
    {{ canvasDimensions?.width }}
  </div>
  <div>canvasRows : {{ canvasRows }} , canvasCols {{ canvasCols }}</div>
  <div>tileHeight : {{ tileHeight }}</div>
  <div>paused : {{ paused }}</div>
  <button @click="addPattern">add pattern</button>
</template>

<script lang="ts"></script>

<style scoped>
#grid {
  background-color: black;
  width: 50%;
  height: 50%;
  aspect-ratio: auto;
}
.toggle {
  /* position: absolute; */
  background-color: blue;
  z-index: 5;
  font-size: 24;
}
</style>
