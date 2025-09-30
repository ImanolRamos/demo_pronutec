<template>
  <div class="final-screen">
    <h1>🎉 Todas las tareas completadas 🎉</h1>
    <p>¡Buen trabajo! Puede revisar los pasos completados o cargar otro CSV para empezar de nuevo.</p>
    <div class="btn-group">
      <button class="btn" @click="$emit('restart')">Reiniciar guía</button>
      <label class="btn" @click="$refs.fileInput.click()">Cargar otro CSV</label>
      <input type="file" ref="fileInput" accept=".csv,text/csv" @change="$emit('file-loaded', $event)" style="display:none" />
    </div>
  </div>
</template>

<script setup>
</script>

<style scoped>
.final-screen { text-align:center; }
.final-screen h1 { font-size:32px; color:#ffd700; margin-bottom:12px; }
.final-screen p { font-size:18px; margin-bottom:20px; }
.btn-group { display:flex; gap:12px; justify-content:center; margin-top:20px; }
.btn { background:#444; color:white; padding:12px 20px; border-radius:8px; cursor:pointer; border:none; }
.btn:hover { background:#666; }
</style>
