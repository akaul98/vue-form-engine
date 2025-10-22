<template>
  <div class="props">
    <h4>Properties</h4>
    <div><strong>Type:</strong> {{ element.type }}</div>

    <div class="field">
      <label>Label</label>
      <input v-model="local.props.label" @input="emitUpdate" />
    </div>

    <div v-if="element.type==='text' || element.type==='textarea'">
      <div class="field">
        <label>Placeholder</label>
        <input v-model="local.props.placeholder" @input="emitUpdate"/>
      </div>
      <div class="field">
        <label>Required</label>
        <input type="checkbox" v-model="local.props.required" @change="emitUpdate"/>
      </div>
      <div class="field">
        <label>Min length</label>
        <input type="number" v-model.number="local.props.minLength" @input="emitUpdate" />
      </div>
      <div class="field">
        <label>Pattern (regex)</label>
        <input v-model="local.props.pattern" @input="emitUpdate"/>
      </div>
    </div>

    <div v-if="element.type==='select' || element.type==='radio'">
      <label>Options (comma separated)</label>
      <input v-model="optionsText" @input="onOptionsChange"/>
    </div>

    <div class="field">
      <label>Position X</label>
      <input type="number" v-model.number="local.x" @input="emitUpdate" />
    </div>
    <div class="field">
      <label>Position Y</label>
      <input type="number" v-model.number="local.y" @input="emitUpdate" />
    </div>

    <div class="actions">
      <button @click="$emit('delete', element.id)">Delete</button>
    </div>
  </div>
</template>

<script setup>
import { reactive, toRefs, watch } from 'vue'
const props = defineProps({ element: Object })
const emit = defineEmits(['update','delete'])

const local = reactive(JSON.parse(JSON.stringify(props.element || {})))

watch(() => props.element, v => {
  Object.assign(local, JSON.parse(JSON.stringify(v)))
})

function emitUpdate() {
  emit('update', JSON.parse(JSON.stringify(local)))
}

let optionsText = (local.props && local.props.options) ? local.props.options.join(',') : ''
watch(() => props.element, () => {
  optionsText = (props.element.props && props.element.props.options) ? props.element.props.options.join(',') : ''
})

function onOptionsChange() {
  if (!local.props) local.props = {}
  local.props.options = optionsText.split(',').map(s => s.trim()).filter(Boolean)
  emitUpdate()
}
</script>

<style scoped>
.props { padding:8px; }
.field { margin-top:8px; display:flex; flex-direction:column; gap:4px; }
.input { width:100% }
.actions { margin-top:12px }
</style>
