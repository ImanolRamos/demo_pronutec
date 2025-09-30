<template>
  <div class="step-card">
    <div class="step-header">
      <div class="step-counter">Paso {{ currentIndex+1 }} de {{ tasks.length }}</div>
      <h1 class="step-title">{{ current.title }}</h1>
      <p class="step-description">{{ current.description }}</p>
      <p v-if="current.control" class="blocking-warning">⚠ Este paso requiere control: "{{ current.control }}"</p>
    </div>

    <div class="timer-container">
      <svg class="progress-ring" viewBox="0 0 42 42">
        <circle cx="21" cy="21" r="15.9155" fill="transparent" stroke="rgba(255,255,255,0.1)" stroke-width="6"/>
        <circle cx="21" cy="21" r="15.9155" fill="transparent"
                :stroke="current.control ? 'orange' : 'green'"
                stroke-width="6"
                stroke-dasharray="100"
                :stroke-dashoffset="100 - progressPercent"
                stroke-linecap="round"
                transform="rotate(-90 21 21)"/>
        <text x="21" y="22.5" font-size="6" text-anchor="middle" fill="white">{{ timeLeft }}s</text>
      </svg>
      <button class="btn-pause" @click="$emit('toggle-pause')">{{ paused ? 'Reanudar' : 'Pausar' }}</button>
    </div>

    <div class="action-container">
      <button v-if="current.control" class="btn-blocking" @click="$emit('press-control')" :disabled="doneThisStep">
        {{ current.control }}
      </button>
      <button v-else class="btn-nonblocking" @click="$emit('complete-step')" :disabled="doneThisStep">
        Marcar completado
      </button>

      <button v-if="current.attachments && current.attachments.length"
              class="btn-attachments"
              @click="$emit('show-attachments')">
        Ver adjuntos ({{ current.attachments.length }})
      </button>
    </div>

    <div class="step-status">Estado: <strong>{{ doneThisStep ? 'Completado' : (paused ? 'Pausado' : 'En curso') }}</strong></div>
  </div>
</template>

<script setup>
import { defineProps } from 'vue'
const props = defineProps({
  current: Object,
  currentIndex: Number,
  tasks: Array,
  timeLeft: Number,
  paused: Boolean,
  progressPercent: Number,
  doneThisStep: Boolean
})
</script>

<style scoped>
.step-card { flex:1; background:#2a2a5a; border-radius:12px; padding:24px; display:flex; flex-direction:column; align-items:center; justify-content:flex-start; overflow-y:auto; }
.step-header { text-align:center; }
.step-title { font-size:28px; color:#ffd700; margin:12px 0; }
.step-description { font-size:18px; margin-bottom:10px; }
.blocking-warning { color:orange; font-weight:bold; }
.timer-container { display:flex; flex-direction:column; align-items:center; margin:20px 0; width:100%; max-width:300px; }
.progress-ring { width:100%; height:auto; max-width:250px; }
.btn-pause { margin-top:12px; background:#888; color:white; font-size:16px; padding:8px 16px; border-radius:6px; cursor:pointer; border:none; }
.action-container { margin-top:20px; display:flex; flex-direction:column; gap:12px; align-items:center; }
.btn-blocking, .btn-nonblocking { font-size:18px; padding:12px 24px; border-radius:8px; border:none; cursor:pointer; }
.btn-blocking { background:orange; color:white; }
.btn-nonblocking { background:green; color:white; }
.btn-attachments { background:#666; color:white; padding:8px 16px; border-radius:6px; border:none; cursor:pointer; }
.step-status { margin-top:16px; font-size:16px; font-weight:bold; }
</style>
