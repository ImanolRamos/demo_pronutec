<template>
  <div class="materials-screen">
    <div class="header">
      <h1>Materiales necesarios</h1>
      <p>Seleccione todos los materiales requeridos antes de continuar.</p>
    </div>

    <div class="upload">
      <label class="upload-label">
        <input type="file" @change="$emit('load-materials', $event)" accept=".csv" />
        Subir CSV de materiales
      </label>
    </div>

    <div v-if="materials.length" class="materials-list">
      <div v-for="m in materials" :key="m.id" class="material-card">
        <label class="material-label">
          <input type="checkbox" v-model="done[String(m.id)]" />
          <span class="material-name">{{ m.name }}</span>
          <span class="material-qty" v-if="m.quantity">x{{ m.quantity }}</span>
        </label>
      </div>
    </div>

    <button class="confirm-btn" :disabled="!allChecked" @click="confirmMaterials">
      Confirmar selección
    </button>
  </div>
</template>


<script setup>
import { reactive, watch, computed } from 'vue'

const props = defineProps({ materials: Array })

// Declarar TODOS los eventos que vamos a emitir
const emit = defineEmits(['load-materials', 'materials-done'])

// Objeto reactivo para checkboxes
const done = reactive({})

// Inicializar done cuando cambien los materiales
watch(
  () => props.materials,
  (newMaterials) => {
    Object.keys(done).forEach(k => delete done[k])
    newMaterials.forEach(m => done[String(m.id)] = false) // todos seleccionados por defecto
  },
  { immediate: true }
)

const allChecked = computed(() =>
  props.materials.length > 0 && props.materials.every(m => done[String(m.id)])
)

function confirmMaterials() {
  const result = props.materials.map(m => [String(m.id), true])
//   console.log('Materiales confirmados:', result)
    emit('materials-done', result)
}
</script>


<style scoped>
.materials-screen {
  width: 600px;
  max-width: 90vw;
  background: #1f1f2e;
  padding: 32px;
  border-radius: 16px;
  box-shadow: 0 12px 30px rgba(0,0,0,0.6);
  display: flex;
  flex-direction: column;
  align-items: center;
  color: #fff;
  font-family: "Segoe UI", sans-serif;
}

.header h1 {
  font-size: 2rem;
  margin-bottom: 8px;
  color: #ffd700;
}
.header p {
  font-size: 1rem;
  color: #ccc;
  margin-bottom: 24px;
  text-align: center;
}

.upload {
  margin-bottom: 24px;
}
.upload-label {
  background: #2a2a5a;
  padding: 12px 24px;
  border-radius: 12px;
  cursor: pointer;
  display: inline-block;
  font-weight: bold;
  transition: background 0.3s;
}
.upload-label:hover {
  background: #3b3b7a;
}
.upload-label input {
  display: none;
}

.materials-list {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  width: 100%;
  margin-bottom: 32px;
}

.material-card {
  background: #2a2a5a;
  border-radius: 12px;
  padding: 16px;
  display: flex;
  align-items: center;
  transition: transform 0.2s, box-shadow 0.2s;
}
.material-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(0,0,0,0.4);
}

.material-label {
  display: flex;
  align-items: center;
  gap: 12px;
  cursor: pointer;
  width: 100%;
}

.material-name {
  flex: 1;
  font-weight: 600;
  font-size: 1rem;
}

.material-qty {
  background: #ffd700;
  color: #1f1f2e;
  padding: 2px 8px;
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: bold;
}

.confirm-btn {
  background: #28a745;
  color: #fff;
  padding: 14px 32px;
  font-size: 1.1rem;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.3s;
}
.confirm-btn:disabled {
  background: #555;
  cursor: not-allowed;
}
.confirm-btn:not(:disabled):hover {
  background: #3ed257;
}
</style>
