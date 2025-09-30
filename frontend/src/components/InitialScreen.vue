<template>
  <div class="initial-screen">
    <h1>Guía Profesional de Operaciones</h1>
    <p>Antes de comenzar, cargue el archivo CSV con los pasos que debe seguir el operario.</p>
    <div class="btn-group">
      <label class="btn" @click="$refs.fileInput.click()">Cargar CSV</label>
      <button class="btn" @click="$emit('use-sample')">Usar CSV de ejemplo</button>
      <input type="file" ref="fileInput" accept=".csv,text/csv" @change="$emit('file-loaded', $event)" style="display:none" />
    </div>
  </div>
</template>

<script setup>
</script>

<style scoped>
.initial-screen { text-align:center; }
.btn-group { display:flex; gap:12px; justify-content:center; margin-top:20px; }
.btn { background:#444; color:white; padding:12px 20px; border-radius:8px; cursor:pointer; border:none; }
.btn:hover { background:#666; }
</style>
