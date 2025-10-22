<template>
  <div class="app">
    <div class="left">
      <Palette @create="createFromPalette"/>
    </div>

    <div class="middle">
      <div class="toolbar">
        <button @click="togglePreview">{{ preview ? 'Edit' : 'Preview' }}</button>
        <button @click="exportJson">Export JSON</button>
        <button @click="importJson">Import JSON</button>
        <button @click="runValidation">Run Validation</button>
      </div>

      <Canvas
        :elements="elements"
        :selectedId="selectedId"
        :preview="preview"
        @update:elements="e => elements = e"
        @select="id => selectedId = id"
        @deselect="() => selectedId = null"
        @move="onMove"
      />
      <div class="validation" v-if="errors.length">
        <h4>Validation Errors</h4>
        <ul>
          <li v-for="err in errors" :key="err.id">{{ err.message }} ({{ err.id }})</li>
        </ul>
      </div>
    </div>

    <div class="right">
      <PropertiesPanel
        v-if="selectedElement"
        :element="selectedElement"
        @update="updateElement"
        @delete="deleteElement"
      />
      <div v-else class="hint">Select an element to edit its properties.</div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import Palette from './components/Palette.vue'
import Canvas from './components/Canvas.vue'
import PropertiesPanel from './components/PropertiesPanel.vue'

const elements = ref([
  // example element
  // {
  //   id: 'el_1', type: 'text', x: 50, y: 20, width: 280, height: 40,
  //   props: { label: 'First name', placeholder: 'Enter name', required: true }
  // }
])
const selectedId = ref(null)
const preview = ref(false)
const errors = ref([])

function createFromPalette(type, x = null, y = null) {
  const id = 'el_' + Date.now()
  
  // Calculate element dimensions first
  const elementWidth = 300
  const elementHeight = type === 'textarea' ? 80 : 40
  
  // Calculate X position: center horizontally in the canvas
  if (x === null) {
    // Calculate available canvas width more accurately
    // Middle column is 60% of screen width, minus gaps and padding
    const screenWidth = window.innerWidth
    const middleColumnWidth = screenWidth * 0.6 // 60% column
    const canvasPadding = 20 // Canvas internal padding (10px each side)
    const availableWidth = middleColumnWidth - canvasPadding
    
    // Center the element within the available canvas space
    x = Math.max(50, (availableWidth - elementWidth) / 2)
  }
  
  // Calculate Y position: find the bottom-most element and add spacing
  if (y === null) {
    if (elements.value.length === 0) {
      y = 40 // First element starts at 40
    } else {
      // Find the element with the highest Y + height position
      const bottomMostY = Math.max(...elements.value.map(el => el.y + el.height))
      y = bottomMostY + 40 // Add 40px spacing after the bottom-most element
    }
  }
  
  const defaultProps = {
    text: { label: 'Text', placeholder: '', required: false },
    textarea: { label: 'Textarea', placeholder: '', required: false },
    checkbox: { label: 'Checkbox', required: false },
    select: { label: 'Select', options: ['Option 1','Option 2'], required: false },
    radio: { label: 'Radio', options: ['A','B'], required: false },
    date: { label: 'Date', required: false },
    button: { label: 'Submit', type: 'submit' }
  }
  elements.value.push({
    id, type, x, y, width: elementWidth, height: elementHeight,
    props: defaultProps[type] || {}
  })
  selectedId.value = id
}

function onMove(updated) {
  // updated is the elements array emitted from Canvas
  elements.value = updated
}

const selectedElement = computed(() => elements.value.find(e => e.id === selectedId.value) || null)

function updateElement(updated) {
  const idx = elements.value.findIndex(e => e.id === updated.id)
  if (idx >= 0) elements.value.splice(idx, 1, { ...elements.value[idx], ...updated })
}

function deleteElement(id) {
  const idx = elements.value.findIndex(e => e.id === id)
  if (idx >= 0) {
    elements.value.splice(idx, 1)
    selectedId.value = null
  }
}

function togglePreview() {
  preview.value = !preview.value
}

function exportJson() {
  const data = JSON.stringify({ elements: elements.value }, null, 2)
  const blob = new Blob([data], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'form.json'
  a.click()
  URL.revokeObjectURL(url)
}

function importJson() {
  const input = document.createElement('input')
  input.type = 'file'
  input.accept = 'application/json'
  input.onchange = async e => {
    const file = e.target.files[0]
    const text = await file.text()
    try {
      const parsed = JSON.parse(text)
      if (Array.isArray(parsed.elements)) elements.value = parsed.elements
    } catch (err) {
      alert('Invalid JSON')
    }
  }
  input.click()
}

function validateElement(element) {
  const p = element.props || {}
  const errs = []
  if (p.required) {
    // for preview runtime, if empty value
    // here we only check presence of label/placeholder as an example
    if (!p.label || !p.label.trim()) errs.push('Label is required')
  }
  if (p.minLength && p.minLength > 0) {
    // meta check example
  }
  // custom pattern check demonstration
  if (p.pattern) {
    try { new RegExp(p.pattern) } catch (_) { errs.push('Invalid regex pattern') }
  }
  return errs
}

function runValidation() {
  errors.value = []
  for (const el of elements.value) {
    const e = validateElement(el)
    if (e.length) errors.value.push({ id: el.id, message: e.join('; ') })
  }
  if (!errors.value.length) alert('No validation issues found')
}
</script>

<style scoped>
.app { 
  display: grid; 
  grid-template-columns: 20% 60% 20%; 
  height: 100vh; 
  gap: 4px; 
  margin: 0;
  padding: 0;
  box-sizing: border-box; 
  overflow: hidden;
}
.left { 
  background: #f6f7fb; 
  padding: 6px; 
  border-radius: 4px;
  overflow-y: auto;
}
.middle { 
  display: flex; 
  flex-direction: column; 
  gap: 4px;
  overflow: hidden;
}
.right { 
  background: #fafafa; 
  padding: 6px; 
  border-radius: 4px;
  overflow-y: auto;
}
.toolbar { 
  display: flex; 
  gap: 6px; 
  flex-shrink: 0;
  padding: 4px 0;
}
.validation { 
  margin-top: 4px; 
  background: #fff7f7; 
  padding: 6px; 
  border-radius: 4px; 
  color: #900; 
}
.hint { 
  padding: 6px; 
  color: #666; 
}
</style>
