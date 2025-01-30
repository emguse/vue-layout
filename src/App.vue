<script setup lang="ts">
import MyCanvas from './components/MyCanvas.vue'
</script>

<template>
  <div>
    <!-- 幅800, 高さ500のキャンバスを描画 -->
    <MyCanvas :width="1100" :height="800" />
  </div>
</template>

<style scoped>
/* 必要に応じてスタイル調整 */
</style>