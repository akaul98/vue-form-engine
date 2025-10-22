<template>
  <div class="palette">
    <h4>Palette</h4>
    <div class="items">
      <div class="item" v-for="t in types" :key="t.type"
           @mousedown.prevent="startCreate(t.type, $event)">
        {{ t.name }}
      </div>
    </div>
    <div class="instructions">Drag from an item or click to create</div>
  </div>
</template>

<script setup>
import { defineEmits } from 'vue' 
const emit = defineEmits(['create'])
const types = [
  { type: 'text', name: 'Text' },
  { type: 'textarea', name: 'Textarea' },
  { type: 'checkbox', name: 'Checkbox' },
  { type: 'select', name: 'Select' },
  { type: 'radio', name: 'Radio' },
  { type: 'date', name: 'Date' },
  { type: 'button', name: 'Button' }
]

let dragState = null

function startCreate(type, e) {
  if (e.type === 'mousedown') {
    // Start drag operation
    dragState = { type, startX: e.clientX, startY: e.clientY, isDragging: false }
    
    // Add event listeners for drag
    document.addEventListener('mousemove', onDragMove)
    document.addEventListener('mouseup', onDragEnd)
    
    // Prevent default to avoid text selection
    e.preventDefault()
  }
}

function onDragMove(e) {
  if (!dragState) return
  
  const deltaX = Math.abs(e.clientX - dragState.startX)
  const deltaY = Math.abs(e.clientY - dragState.startY)
  
  // Start dragging if moved more than 5px
  if (deltaX > 5 || deltaY > 5) {
    dragState.isDragging = true
    // Could add visual feedback here (drag preview)
  }
}

function onDragEnd(e) {
  if (!dragState) return
  
  // Clean up event listeners
  document.removeEventListener('mousemove', onDragMove)
  document.removeEventListener('mouseup', onDragEnd)
  
  if (dragState.isDragging) {
    // Get canvas element bounds to calculate relative position
    const canvas = document.querySelector('.canvas-area')
    if (canvas) {
      const canvasRect = canvas.getBoundingClientRect()
      const relativeX = e.clientX - canvasRect.left
      const relativeY = e.clientY - canvasRect.top
      
      // Only create if dropped within canvas bounds
      if (relativeX >= 0 && relativeY >= 0 && 
          relativeX <= canvasRect.width && relativeY <= canvasRect.height) {
        emit('create', dragState.type, relativeX, relativeY)
      }
    }
  } else {
    // Just a click, not a drag - use default positioning
    emit('create', dragState.type)
  }
  
  dragState = null
}
</script>

<style scoped>
.palette { padding:8px; }
.item { padding:8px; margin:6px 0; cursor:pointer; background:#fff; border-radius:4px; box-shadow:0 1px 2px rgba(0,0,0,.04) }
.instructions { margin-top:8px; color:#666; font-size:12px }
</style>
