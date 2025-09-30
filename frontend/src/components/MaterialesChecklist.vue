<template>
  <div class="materials-checklist">
    <h1>Selecciona los materiales a usar</h1>
    <input type="file" @change="onFile" accept=".csv" />
    <ul>
      <li v-for="mat in materials" :key="mat.id">
        <label>
          <input type="checkbox" v-model="mat._done" />
          {{ mat.name }}
        </label>
      </li>
    </ul>
    <button class="btn-confirm" @click="confirmMaterials" :disabled="materials.length === 0">
      Confirmar materiales
    </button>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue'

const props = defineProps({
  materials: Array
})

const emit = defineEmits(['file-loaded','done'])

function onFile(e){
  emit('file-loaded', e)
}

function confirmMaterials(){
  emit('done')
}
</script>

<style scoped>
.materials-checklist { background:#2a2a5a; padding:30px; border-radius:12px; width:500px; max-width:90%; text-align:center; color:white; }
.materials-checklist h1 { font-size:28px; margin-bottom:20px; color:#ffd700; }
.materials-checklist ul { list-style:none; padding:0; max-height:300px; overflow-y:auto; margin-bottom:20px; }
.materials-checklist li { margin-bottom:10px; font-size:18px; }
.btn-confirm { padding:12px 24px; font-size:18px; background:green; color:white; border:none; border-radius:8px; cursor:pointer; }
.btn-confirm:disabled { background:gray; cursor:not-allowed; }
</style>
