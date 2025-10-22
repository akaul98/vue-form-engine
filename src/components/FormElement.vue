<template>
  <div
    class="form-element"
    :class="{ selected }"
    :style="style"
    @pointerdown.stop="onPointerDown"
    ref="elRef"
  >
    <template v-if="preview">
      <!-- Render actual control in preview -->
      <label v-if="element.props?.label">{{ element.props.label }}</label>
      <input v-if="element.type === 'text'" :placeholder="element.props.placeholder" />
      <textarea v-if="element.type === 'textarea'" :placeholder="element.props.placeholder"></textarea>
      <div v-if="element.type === 'checkbox'">
        <input type="checkbox" :id="element.id" /> <label :for="element.id">{{ element.props.label }}</label>
      </div>
      <select v-if="element.type === 'select'">
        <option v-for="opt in element.props.options" :key="opt">{{ opt }}</option>
      </select>
      <button v-if="element.type === 'button'">{{ element.props.label }}</button>
    </template>

    <template v-else>
      <!-- Editing view: show a simple wireframe representing the control -->
      <div class="wire">
        <div class="label">{{ element.props?.label || element.type }}</div>
        <div class="control" v-if="element.type === 'text'">[ text input ]</div>
        <div class="control" v-if="element.type === 'textarea'">[ textarea ]</div>
        <div class="control" v-if="element.type === 'checkbox'">[ checkbox ]</div>
        <div class="control" v-if="element.type === 'select'">[ select ]</div>
        <div class="control" v-if="element.type === 'radio'">[ radio ]</div>
        <div class="control" v-if="element.type === 'date'">[ date ]</div>
        <div class="control" v-if="element.type === 'button'">[ button ]</div>
      </div>
    </template>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
const props = defineProps({
  element: { type: Object, required: true },
  selected: { type: Boolean, default: false },
  preview: { type: Boolean, default: false }
})
const emit = defineEmits(['select','startDrag','move','update'])

const elRef = ref(null)
const dragging = ref(false)

const style = computed(() => ({
  position: 'absolute',
  left: props.element.x + 'px',
  top: props.element.y + 'px',
  width: props.element.width + 'px',
  height: props.element.height + 'px',
  cursor: props.preview ? 'default' : 'grab',
  boxSizing: 'border-box'
}))

function onPointerDown(e) {
  emit('select', props.element.id)
  if (props.preview) return
  // begin drag
  const rect = elRef.value.getBoundingClientRect()
  const offsetX = props.element.x
  const offsetY = props.element.y
  dragging.value = true
  elRef.value.setPointerCapture(e.pointerId)
  emit('startDrag', {
    id: props.element.id,
    startX: e.clientX,
    startY: e.clientY,
    offsetX, offsetY
  })
}
</script>

<style scoped>
.form-element { 
  border: 1px solid transparent; 
  background: rgba(0,0,0,0.02); 
  padding: 6px; 
  border-radius: 4px;
  margin-bottom: 20px;
}
.form-element.selected { border-color: #2b6ef6; box-shadow: 0 6px 18px rgba(43,110,246,0.12) }
.wire { font-size:13px; color:#333; }
.label { font-weight:600; margin-bottom:4px; }
.control { color:#666; font-size:12px; }
</style>
