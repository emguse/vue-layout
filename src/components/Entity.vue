<template>
  <div
    ref="entityRef"
    class="entity"
    :style="entityStyle"
    @click="handleClick"
    @dblclick="handleDoubleClick"
  >
    <span class="entity-label">{{ entity.entityName }}</span>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, computed, watch } from 'vue';
import interact from 'interactjs';
import { EntityData } from './Canvas.vue';

export default defineComponent({
  name: 'Entity',
  props: {
    entity: {
      type: Object as () => EntityData,
      required: true,
    },
    gridSpacing: {
      type: Number,
      required: true,
    },
    zoom: {
      type: Number,
      required: true,
    },
  },
  emits: ['update', 'dblclick', 'select'],
  setup(props, { emit }) {
    const entityRef = ref<HTMLElement | null>(null);

    const entityStyle = computed(() => ({
      position: 'absolute',
      left: props.entity.anchorX + 'px',
      top: props.entity.anchorY + 'px',
      width: props.entity.sizeX + 'px',
      height: props.entity.sizeY + 'px',
      border: `2px solid ${props.entity.lineColor}`,
      backgroundColor: props.entity.fillColor,
      transform: `rotate(${props.entity.rotation}deg)`,
      cursor: 'move',
      boxSizing: 'border-box',
    }));

    // Initial Drag Settings
    const initializeDraggable = () => {
      if (entityRef.value) {
        interact(entityRef.value).draggable({
          modifiers: [
            // Snap: to grid intersection (relativePoints: {0,0})
            interact.modifiers.snap({
              targets: [
                interact.snappers.grid({ x: props.gridSpacing, y: props.gridSpacing })
              ],
              range: props.gridSpacing,
              relativePoints: [{ x: 0, y: 0 }],
            }),
            // restrict: Restrict to parent element
            interact.modifiers.restrict({
              restriction: 'parent',
              elementRect: { top: 0, left: 0, bottom: 1, right: 1 },
              endOnly: true,
            })
          ],
          listeners: {
            move(event) {
              // Zoom and move
              props.entity.anchorX += event.dx / props.zoom;
              props.entity.anchorY += event.dy / props.zoom;
              emit('update', { ...props.entity });
            },
            end(event) {
              // When dragging ends, the grid offset (-9, +3) is corrected.
              props.entity.anchorX = Math.round(props.entity.anchorX) + 9;
              props.entity.anchorY = Math.round(props.entity.anchorY) - 3;
              emit('update', { ...props.entity });
            }
          },
        });
      }
    };

    onMounted(() => {
      initializeDraggable();
    });

    // gridSpacing 変更時に、snap modifier の設定を再適用
    watch(() => props.gridSpacing, (newSpacing) => {
      if (entityRef.value) {
        interact(entityRef.value).draggable({
          modifiers: [
            interact.modifiers.snap({
              targets: [
                interact.snappers.grid({ x: newSpacing, y: newSpacing })
              ],
              range: newSpacing,
              relativePoints: [{ x: 0, y: 0 }],
            }),
            interact.modifiers.restrict({
              restriction: 'parent',
              elementRect: { top: 0, left: 0, bottom: 1, right: 1 },
              endOnly: true,
            })
          ]
        });
      }
    });

    const handleDoubleClick = () => {
      emit('dblclick', { ...props.entity });
    };

    const handleClick = () => {
      emit('select', { ...props.entity });
    };

    return { entityRef, entityStyle, handleDoubleClick, handleClick };
  },
});
</script>

<style scoped>
.entity {
  user-select: none;
}
.entity-label {
  position: absolute;
  bottom: 0;
  left: 0;
  background: rgba(255, 255, 255, 0.7);
  font-size: 12px;
  padding: 2px;
}
</style>