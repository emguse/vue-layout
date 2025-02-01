<template>
  <div id="app">
    <GlobalNav
      :zoom="zoom"
      @downloadEntities="downloadEntities"
      @projectDefinition="openProjectDefinition"
      @addEntity="addNewEntity"
      @copyEntity="copyEntity"
      @setZoom="setZoomValue"
    />
    <Canvas ref="canvasRef" :initialZoom="zoom" />
    <ProjectDefinition
      v-if="showProjectDefinition"
      :initialWidth="canvasWidth"
      :initialHeight="canvasHeight"
      :initialGridSpacing="gridSpacing"
      @update="updateProjectDefinition"
      @close="showProjectDefinition = false"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import GlobalNav from './components/GlobalNav.vue';
import Canvas from './components/Canvas.vue';
import ProjectDefinition from './components/ProjectDefinition.vue';

export default defineComponent({
  name: 'App',
  components: { GlobalNav, Canvas, ProjectDefinition },
  setup() {
    const canvasRef = ref<any>(null);
    const showProjectDefinition = ref(false);
    const canvasWidth = ref(2000);
    const canvasHeight = ref(2000);
    const gridSpacing = ref(50);
    const zoom = ref(1);

    const downloadEntities = () => {
      if (canvasRef.value) {
        const entities = canvasRef.value.getEntities();
        const dataStr =
          'data:text/json;charset=utf-8,' +
          encodeURIComponent(JSON.stringify(entities, null, 2));
        const downloadAnchorNode = document.createElement('a');
        downloadAnchorNode.setAttribute('href', dataStr);
        downloadAnchorNode.setAttribute('download', 'entities.json');
        document.body.appendChild(downloadAnchorNode);
        downloadAnchorNode.click();
        downloadAnchorNode.remove();
      }
    };

    const openProjectDefinition = () => {
      showProjectDefinition.value = true;
    };

    const updateProjectDefinition = (data: { width: number; height: number; gridSpacing: number }) => {
      if (canvasRef.value) {
        canvasRef.value.setCanvasSize(data.width, data.height);
        canvasRef.value.setGridSpacing(data.gridSpacing);
      }
      canvasWidth.value = data.width;
      canvasHeight.value = data.height;
      gridSpacing.value = data.gridSpacing;
      showProjectDefinition.value = false;
    };

    const addNewEntity = () => {
      if (canvasRef.value) {
        canvasRef.value.addEntity();
      }
    };

    const copyEntity = () => {
      if (canvasRef.value) {
        canvasRef.value.copySelectedEntity();
      }
    };

    const setZoomValue = (newZoom: number) => {
      zoom.value = newZoom;
      if (canvasRef.value) {
        canvasRef.value.setZoom(newZoom);
      }
    };

    return {
      canvasRef,
      downloadEntities,
      openProjectDefinition,
      showProjectDefinition,
      canvasWidth,
      canvasHeight,
      gridSpacing,
      zoom,
      updateProjectDefinition,
      addNewEntity,
      copyEntity,
      setZoomValue,
    };
  },
});
</script>