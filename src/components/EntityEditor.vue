<!-- components/EntityEditor.vue -->
<template>
  <div class="modal-overlay">
    <div class="modal-content">
      <h3>Edit Entity Properties</h3>
      <form @submit.prevent="submitForm">
        <div>
          <label>Entity Name:</label>
          <input type="text" v-model="localEntity.entityName" />
        </div>
        <div>
          <label>Entity Type:</label>
          <input type="text" v-model="localEntity.entityType" />
        </div>
        <div>
          <label>Anchor X:</label>
          <input type="number" v-model.number="localEntity.anchorX" />
        </div>
        <div>
          <label>Anchor Y:</label>
          <input type="number" v-model.number="localEntity.anchorY" />
        </div>
        <div>
          <label>Rotation (deg):</label>
          <input type="number" v-model.number="localEntity.rotation" />
        </div>
        <div>
          <label>Size X:</label>
          <input type="number" v-model.number="localEntity.sizeX" />
        </div>
        <div>
          <label>Size Y:</label>
          <input type="number" v-model.number="localEntity.sizeY" />
        </div>
        <div>
          <label>Line Color:</label>
          <input type="color" v-model="localEntity.lineColor" />
        </div>
        <div>
          <label>Fill Color:</label>
          <input type="color" v-model="localEntity.fillColor" />
        </div>
        <div>
          <label>Additional Attributes (JSON):</label>
          <textarea v-model="additionalAttributesText" rows="4" cols="30"></textarea>
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
import { defineComponent, ref, watch } from 'vue';
import { EntityData } from './Canvas.vue';

export default defineComponent({
  name: 'EntityEditor',
  props: {
    entity: {
      type: Object as () => EntityData,
      required: true,
    },
  },
  emits: ['update', 'close'],
  setup(props, { emit }) {
    const localEntity = ref({ ...props.entity });
    const additionalAttributesText = ref(
      JSON.stringify(props.entity.additionalAttributes, null, 2)
    );

    watch(
      () => props.entity,
      (newVal) => {
        localEntity.value = { ...newVal };
        additionalAttributesText.value = JSON.stringify(newVal.additionalAttributes, null, 2);
      }
    );

    const submitForm = () => {
      try {
        localEntity.value.additionalAttributes = JSON.parse(additionalAttributesText.value);
      } catch (error) {
        alert('Invalid JSON in additional attributes');
        return;
      }
      // Coordinates are rounded to integers
      localEntity.value.anchorX = Math.round(localEntity.value.anchorX);
      localEntity.value.anchorY = Math.round(localEntity.value.anchorY);
      emit('update', localEntity.value);
    };

    return { localEntity, additionalAttributesText, submitForm };
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