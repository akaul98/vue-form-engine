<template>
  <div class="canvas" ref="wrap" @pointerdown="onCanvasPointerDown">
    <!-- optional grid -->
    <div class="canvas-area" ref="area">
      <FormElement
        v-for="el in localElements"
        :key="el.id"
        :element="el"
        :selected="selectedId === el.id"
        :preview="preview"
        @select="select"
        @startDrag="startDrag"
        @move="onElementMove"
        @update="updateLocalElement"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, watch, reactive } from 'vue'
import FormElement from './FormElement.vue'
const props = defineProps({
  elements: { type: Array, required: true },
  selectedId: { type: [String, null], default: null },
  preview: { type: Boolean, default: false }
})
const emit = defineEmits(['update:elements','select','deselect','move'])

const localElements = ref(JSON.parse(JSON.stringify(props.elements || [])))
watch(()=>props.elements, v => localElements.value = JSON.parse(JSON.stringify(v)), { deep: true })

const wrap = ref(null)
const area = ref(null)

function select(id) {
  emit('select', id)
}

function onCanvasPointerDown(e) {
  // clicking canvas deselects
  if (e.target === wrap.value || e.target === area.value) {
    emit('deselect')
  }
}

// Dragging logic: when FormElement emits startDrag, we attach pointermove at document
let dragState = null
function startDrag({ id, startX, startY, offsetX, offsetY }) {
  if (props.preview) return
  dragState = { id, startX, startY, offsetX, offsetY }
  document.addEventListener('pointermove', pointerMove)
  document.addEventListener('pointerup', pointerUp, { once: true })
}

function pointerMove(e) {
  if (!dragState) return
  const el = localElements.value.find(x => x.id === dragState.id)
  if (!el) return
  const dx = e.clientX - dragState.startX
  const dy = e.clientY - dragState.startY
  el.x = dragState.offsetX + dx
  el.y = dragState.offsetY + dy
  emit('update:elements', localElements.value)
  emit('move', localElements.value)
}

function pointerUp() {
  dragState = null
  document.removeEventListener('pointermove', pointerMove)
}

function onElementMove(updated) {
  const idx = localElements.value.findIndex(x => x.id === updated.id)
  if (idx >= 0) localElements.value.splice(idx, 1, updated)
  emit('update:elements', localElements.value)
  emit('move', localElements.value)
}

function updateLocalElement(updated) {
  const idx = localElements.value.findIndex(x => x.id === updated.id)
  if (idx >= 0) localElements.value.splice(idx, 1, updated)
  emit('update:elements', localElements.value)
}
</script>

<style scoped>
.canvas { 
  background: #fff; 
  border: 1px dashed #e1e6f0; 
  border-radius: 6px; 
  padding: 10px; 
  height: 100%; 
  position: relative; 
  overflow: auto;
}
.canvas-area { 
  position: relative; 
  min-height: 100%; 
  padding-bottom: 40px;
}
</style>
