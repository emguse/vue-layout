<template>
  <div class="canvas-container" ref="containerRef" @wheel.prevent="onWheel">
    <!-- A wrapper around the canvas content: its size is the canvas size x zoom factor. -->
    <div class="canvas-wrapper" :style="wrapperStyle">
      <div class="canvas-content" :style="canvasStyle">
        <GridLayer
          :gridSpacing="gridSpacing"
          :canvasWidth="canvasWidth"
          :canvasHeight="canvasHeight"
        />
        <Entity
          v-for="entity in entities"
          :key="entity.id"
          :entity="entity"
          :gridSpacing="gridSpacing"
          :zoom="zoom"
          @update="updateEntity"
          @dblclick="onEntityDoubleClick"
          @select="onEntityClick"
        />
      </div>
    </div>
  </div>
  <EntityEditor
    v-if="selectedEntity"
    :entity="selectedEntity"
    @update="onEntityUpdate"
    @close="selectedEntity = null"
  />
</template>

<script lang="ts">
import { defineComponent, ref, reactive, onMounted, computed } from 'vue';
import GridLayer from './GridLayer.vue';
import Entity from './Entity.vue';
import EntityEditor from './EntityEditor.vue';
import { v4 as uuidv4 } from 'uuid';

export interface EntityData {
  id: string;
  anchorX: number;
  anchorY: number;
  rotation: number;
  sizeX: number;
  sizeY: number;
  lineColor: string;
  fillColor: string;
  entityName: string;
  entityType: string;
  additionalAttributes: Record<string, any>;
}

export default defineComponent({
  name: 'Canvas',
  components: { GridLayer, Entity, EntityEditor },
  props: {
    initialZoom: { type: Number, default: 1 },
  },
  setup(props, { expose }) {
    const entities = reactive<EntityData[]>([]);
    const zoom = ref(props.initialZoom);
    const gridSpacing = ref(50);
    const canvasWidth = ref(2000);
    const canvasHeight = ref(2000);
    const containerRef = ref<HTMLElement | null>(null);
    // Keep selected (last clicked) entity
    const selectedEntity = ref<EntityData | null>(null);

    const wrapperStyle = computed(() => ({
      width: canvasWidth.value * zoom.value + 'px',
      height: canvasHeight.value * zoom.value + 'px',
    }));

    const canvasStyle = computed(() => ({
      transform: `scale(${zoom.value})`,
      transformOrigin: '0 0',
      width: canvasWidth.value + 'px',
      height: canvasHeight.value + 'px',
      position: 'relative',
      boxSizing: 'border-box',
    }));

    const loadDefaultEntities = async () => {
      try {
        const response = await fetch('/default-entities.json');
        if (response.ok) {
          const data = await response.json();
          data.forEach((item: Partial<EntityData>) => {
            entities.push({
              id: item.id || uuidv4(),
              anchorX: item.anchorX ?? 0,
              anchorY: item.anchorY ?? 0,
              rotation: item.rotation ?? 0,
              sizeX: item.sizeX ?? 100,
              sizeY: item.sizeY ?? 100,
              lineColor: item.lineColor || '#000000',
              fillColor: item.fillColor || '#ff0000',
              entityName: item.entityName || 'Unnamed',
              entityType: item.entityType || 'Default',
              additionalAttributes: item.additionalAttributes || {},
            });
          });
        }
      } catch (error) {
        console.error('Failed to load default entities:', error);
      }
    };

    onMounted(() => {
      loadDefaultEntities();
    });

    // Zoom change by Ctrl+Wheel (UI operation is also possible)
    const onWheel = (event: WheelEvent) => {
      if (event.ctrlKey) {
        const delta = event.deltaY;
        if (delta < 0) {
          zoom.value *= 1.1;
        } else {
          zoom.value /= 1.1;
        }
      }
    };

    const updateEntity = (updatedEntity: EntityData) => {
      const index = entities.findIndex(e => e.id === updatedEntity.id);
      if (index !== -1) {
        updatedEntity.anchorX = Math.round(updatedEntity.anchorX);
        updatedEntity.anchorY = Math.round(updatedEntity.anchorY);
        entities[index] = { ...updatedEntity };
      }
    };

    // Public methods
    const getEntities = () => entities;
    const setCanvasSize = (width: number, height: number) => {
      canvasWidth.value = width;
      canvasHeight.value = height;
    };
    const setGridSpacing = (spacing: number) => {
      gridSpacing.value = spacing;
    };
    const setZoom = (newZoom: number) => {
      zoom.value = newZoom;
    };
    const addEntity = () => {
      const newEntity: EntityData = {
        id: uuidv4(),
        anchorX: 0,
        anchorY: 0,
        rotation: 0,
        sizeX: 100,
        sizeY: 100,
        lineColor: '#000000',
        fillColor: '#ff0000',
        entityName: 'New Entity',
        entityType: 'Default',
        additionalAttributes: {},
      };
      entities.push(newEntity);
    };
    const copySelectedEntity = () => {
      if (selectedEntity.value) {
        const newEntity = { ...selectedEntity.value, id: uuidv4() };
        entities.push(newEntity);
      }
    };

    // Exposing public methods
    expose({ getEntities, setCanvasSize, setGridSpacing, setZoom, addEntity, copySelectedEntity });

    // Update selection state when clicking or double-clicking an entity
    const onEntityClick = (entity: EntityData) => {
      selectedEntity.value = { ...entity };
    };
    const onEntityDoubleClick = (entity: EntityData) => {
      selectedEntity.value = { ...entity };
    };
    const onEntityUpdate = (updatedEntity: EntityData) => {
      updatedEntity.anchorX = Math.round(updatedEntity.anchorX);
      updatedEntity.anchorY = Math.round(updatedEntity.anchorY);
      updateEntity(updatedEntity);
      selectedEntity.value = null;
    };

    return {
      entities,
      zoom,
      gridSpacing,
      canvasWidth,
      canvasHeight,
      containerRef,
      wrapperStyle,
      canvasStyle,
      updateEntity,
      selectedEntity,
      onEntityDoubleClick,
      onEntityUpdate,
      onWheel,
      onEntityClick
    };
  },
});
</script>

<style scoped>
/* The canvas container displays scroll bars on the right and bottom edges of the screen (placed full screen below the GlobalNav) */
.canvas-container {
  position: absolute;
  top: 100px; /* GlobalNav height (example) */
  left: 0;
  right: 0;
  bottom: 0;
  overflow: auto;
  background-color: #f0f0f0;
}

.canvas-content {
  background-color: white;
  border: 1px solid #ccc;
  position: relative;
  box-sizing: border-box;
}
</style>