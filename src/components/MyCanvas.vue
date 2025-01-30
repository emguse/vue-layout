<script setup lang="ts">
import { ref, onMounted, defineProps } from 'vue';

// ------------ Type definition ------------
interface Rectangle {
  x: number;
  y: number;
  w: number;
  h: number;
}

// ------------ Props ------------
const props = defineProps<{
  width?: number;
  height?: number;
}>();

// Canvas size received from props. ?? default 
const canvasWidth = props.width ?? 600;
const canvasHeight = props.height ?? 400;

// ------------ Reactive variables ------------
const canvasRef = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D | null = null; // 2D Drawing Context

const rectangles = ref<Rectangle[]>([]);

// Is a new rectangle being drawn?
const isDrawing = ref(false);
const startX = ref(0);
const startY = ref(0);
const currentX = ref(0);
const currentY = ref(0);

// For dragging an existing rectangle
const isDraggingRect = ref(false);
const draggingRectIndex = ref<number | null>(null);
const dragOffsetX = ref(0);
const dragOffsetY = ref(0);

// ------------ Methods ------------
function drawAll() {
  // null guards
  if (!ctx) return;
  
  // set to local
  const localCtx = ctx;

  localCtx.clearRect(0, 0, canvasWidth, canvasHeight);

  rectangles.value.forEach((rect) => {
    localCtx.strokeStyle = 'blue';
    localCtx.lineWidth = 2;
    localCtx.strokeRect(rect.x, rect.y, rect.w, rect.h);
  });
}

function drawPreviewRect() {
  if (!ctx) return;
  ctx.strokeStyle = 'red';
  ctx.lineWidth = 2;
  const w = currentX.value - startX.value;
  const h = currentY.value - startY.value;
  ctx.strokeRect(startX.value, startY.value, w, h);
}

function getRectIndexUnderCursor(x: number, y: number): number {
  // Search from the back (front) and prioritize the frontmost rectangle
  for (let i = rectangles.value.length - 1; i >= 0; i--) {
    const rect = rectangles.value[i];
    if (
      x >= rect.x &&
      x <= rect.x + rect.w &&
      y >= rect.y &&
      y <= rect.y + rect.h
    ) {
      return i;
    }
  }
  return -1;
}

function getMousePos(e: MouseEvent) {
  if (!canvasRef.value) {
    return { offsetX: 0, offsetY: 0 };
  }
  const rect = canvasRef.value.getBoundingClientRect();
  return {
    offsetX: e.clientX - rect.left,
    offsetY: e.clientY - rect.top,
  };
}

// ------------ Event Handlers ------------
function handleMouseDown(e: MouseEvent) {
  const { offsetX, offsetY } = getMousePos(e);

  // If on an existing rectangle, start dragging the rectangle
  const rectIndex = getRectIndexUnderCursor(offsetX, offsetY);
  if (rectIndex !== -1) {
    isDraggingRect.value = true;
    draggingRectIndex.value = rectIndex;

    // Record the offset of the drag start point
    const rect = rectangles.value[rectIndex];
    dragOffsetX.value = offsetX - rect.x;
    dragOffsetY.value = offsetY - rect.y;
    return;
  }

  // Otherwise, new drawing mode
  isDrawing.value = true;
  startX.value = offsetX;
  startY.value = offsetY;
  currentX.value = offsetX;
  currentY.value = offsetY;
}

function handleMouseMove(e: MouseEvent) {
  const { offsetX, offsetY } = getMousePos(e);

  // Dragging an existing rectangle
  if (isDraggingRect.value && draggingRectIndex.value !== null) {
    const rect = rectangles.value[draggingRectIndex.value];
    rect.x = offsetX - dragOffsetX.value;
    rect.y = offsetY - dragOffsetY.value;
    drawAll();
    return;
  }

  // In new drawing mode
  if (isDrawing.value) {
    currentX.value = offsetX;
    currentY.value = offsetY;
    drawAll();
    drawPreviewRect();
  }
}

function handleMouseUp() {
  // Drag End
  if (isDraggingRect.value) {
    isDraggingRect.value = false;
    draggingRectIndex.value = null;
    return;
  }

  // New drawing completed
  if (isDrawing.value) {
    isDrawing.value = false;
    const w = currentX.value - startX.value;
    const h = currentY.value - startY.value;
    rectangles.value.push({ x: startX.value, y: startY.value, w, h });
  }
}

function handleMouseLeave() {
  // Once you leave the canvas, all modes are cleared.
  isDrawing.value = false;
  isDraggingRect.value = false;
  draggingRectIndex.value = null;
}

// ------------ Initial processing at mount ------------
onMounted(() => {
  if (canvasRef.value) {
    ctx = canvasRef.value.getContext('2d');
  }
  drawAll();
});

// ------------ If you want to handle size changes with watch ------------
/*
watch(() => props.width, () => {
// If you want to do something when the canvas size changes
// Example: rectangles.value = []; etc.
});
watch(() => props.height, () => {
// Same as above
});
*/
</script>

<template>
  <canvas
    ref="canvasRef"
    :width="canvasWidth"
    :height="canvasHeight"
    style="border: 1px solid #333;"
    @mousedown="handleMouseDown"
    @mousemove="handleMouseMove"
    @mouseup="handleMouseUp"
    @mouseleave="handleMouseLeave"
  ></canvas>
</template>

<style scoped>
/* Define styles as needed */
</style>