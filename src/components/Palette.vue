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

function startCreate(type, e) {
  // Let the parent component handle the positioning logic
  emit('create', type)
}
</script>

<style scoped>
.palette { padding:8px; }
.item { padding:8px; margin:6px 0; cursor:pointer; background:#fff; border-radius:4px; box-shadow:0 1px 2px rgba(0,0,0,.04) }
.instructions { margin-top:8px; color:#666; font-size:12px }
</style>
