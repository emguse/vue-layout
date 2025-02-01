<template>
    <div class="modal-overlay">
      <div class="modal-content">
        <h3>Project Definition</h3>
        <form @submit.prevent="submitForm">
          <div>
            <label>Canvas Width:</label>
            <input type="number" v-model.number="canvasWidth" required />
          </div>
          <div>
            <label>Canvas Height:</label>
            <input type="number" v-model.number="canvasHeight" required />
          </div>
          <div>
            <label>Grid Spacing:</label>
            <input type="number" v-model.number="gridSpacing" required />
          </div>
          <div class="modal-buttons">
            <button type="submit">Save</button>
            <button type="button" @click="$emit('close')">Cancel</button>
          </div>
        </form>
      </div>
    </div>
  </template>
  
  <script lang="ts">
  import { defineComponent, ref } from 'vue';
  export default defineComponent({
    name: 'ProjectDefinition',
    props: {
      initialWidth: { type: Number, required: true },
      initialHeight: { type: Number, required: true },
      initialGridSpacing: { type: Number, required: true },
    },
    emits: ['update', 'close'],
    setup(props, { emit }) {
      const canvasWidth = ref(props.initialWidth);
      const canvasHeight = ref(props.initialHeight);
      const gridSpacing = ref(props.initialGridSpacing);
      const submitForm = () => {
        emit('update', {
          width: canvasWidth.value,
          height: canvasHeight.value,
          gridSpacing: gridSpacing.value,
        });
      };
      return { canvasWidth, canvasHeight, gridSpacing, submitForm };
    },
  });
  </script>
  
  <style scoped>
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .modal-content {
    background: white;
    padding: 20px;
    border-radius: 4px;
    min-width: 300px;
  }
  .modal-buttons {
    margin-top: 10px;
    display: flex;
    justify-content: space-between;
  }
  </style>