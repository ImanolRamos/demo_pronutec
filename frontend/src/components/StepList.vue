<template>
  <div class="steps-panel">
    <h3>Pasos</h3>
    <div class="step-list">
      <div v-for="(t,i) in steps" :key="t.id"
           :class="['step-item', {active: i + start === currentIndex, done: (i + start) < currentIndex}]"
           :ref="el => stepRefs[i + start] = el">
        <div class="step-number">{{ i + start + 1 }}</div>
        <div class="step-title-small">{{ t.title }}</div>
        <div class="step-duration-small">
          {{ t.duration }}s 
          <span v-if="t.control">· {{ t.control }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { defineProps } from 'vue'
const props = defineProps({
  steps: Array,
  currentIndex: Number,
  start: { type:Number, default:0 },
  stepRefs: Array
})
</script>

<style scoped>
.steps-panel { width:250px; background:#2a2a2a; border-radius:10px; padding:12px; display:flex; flex-direction:column; }
.step-list { display:flex; flex-direction:column; gap:6px; flex:1; overflow:hidden; }
.step-item { padding:6px 8px; border-radius:6px; background:#3a3a3a; color:#ccc; display:flex; flex-direction:column; }
.step-item.active { background:#4444aa; color:white; font-weight:bold; box-shadow:0 0 8px #888; }
.step-item.done { background:#555; color:#999; }
</style>
