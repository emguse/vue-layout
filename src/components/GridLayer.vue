<!-- components/GridLayer.vue -->
<template>
  <svg :width="canvasWidth" :height="canvasHeight" class="grid-layer">
    <!-- Render vertical grid lines -->
    <template v-for="x in verticalLines" :key="'v' + x">
      <line :x1="x" y1="0" :x2="x" :y2="canvasHeight" stroke="#e0e0e0" />
    </template>
    <!-- Render horizontal grid lines -->
    <template v-for="y in horizontalLines" :key="'h' + y">
      <line x1="0" :y1="y" :x2="canvasWidth" :y2="y" stroke="#e0e0e0" />
    </template>
  </svg>
</template>

<script lang="ts">
import { defineComponent, computed } from 'vue';

export default defineComponent({
  name: 'GridLayer',
  props: {
    gridSpacing: {
      type: Number,
      required: true,
    },
    canvasWidth: {
      type: Number,
      required: true,
    },
    canvasHeight: {
      type: Number,
      required: true,
    },
  },
  setup(props) {
    // Create an array of x positions for vertical lines.
    const verticalLines = computed(() => {
      const lines = [];
      for (let x = 0; x <= props.canvasWidth; x += props.gridSpacing) {
        lines.push(x);
      }
      return lines;
    });

    // Create an array of y positions for horizontal lines.
    const horizontalLines = computed(() => {
      const lines = [];
      for (let y = 0; y <= props.canvasHeight; y += props.gridSpacing) {
        lines.push(y);
      }
      return lines;
    });

    return { verticalLines, horizontalLines };
  },
});
</script>

<style scoped>
.grid-layer {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none; /* Allows click events to pass through */
}
</style>