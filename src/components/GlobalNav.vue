<template>
  <nav class="global-nav">
    <h1>Layout PoC</h1>
    <div class="nav-controls">
      <div class="zoom-control">
        <label>Zoom:</label>
        <input
          type="range"
          min="0.1"
          max="3"
          step="0.1"
          :value="zoom"
          @input="onZoomChange($event)"
        />
        <span>{{ zoom }}</span>
      </div>
      <button @click="$emit('projectDefinition')">Project Definition</button>
      <button @click="$emit('addEntity')">Add Entity</button>
      <button @click="$emit('copyEntity')">Copy Entity</button>
      <button @click="$emit('downloadEntities')">Download JSON</button>
    </div>
  </nav>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
export default defineComponent({
  name: 'GlobalNav',
  props: {
    zoom: {
      type: Number,
      required: true,
    }
  },
  emits: ['projectDefinition', 'addEntity', 'copyEntity', 'downloadEntities', 'setZoom'],
  setup(props, { emit }) {
    // Emit new zoom value when slider changes.
    const onZoomChange = (event: Event) => {
      const target = event.target as HTMLInputElement;
      const newZoom = parseFloat(target.value);
      emit('setZoom', newZoom);
    };
    return { onZoomChange };
  }
});
</script>

<style scoped>
.global-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #333;
  color: white;
  padding: 10px;
}
.nav-controls {
  display: flex;
  align-items: center;
}
.zoom-control {
  display: flex;
  align-items: center;
  margin-right: 10px;
}
.zoom-control input {
  margin: 0 5px;
}
.global-nav button {
  margin-left: 10px;
}
</style>