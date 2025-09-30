<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal-content">
      <h2>Adjuntos de "{{ title }}"</h2>
      <ul>
        <li v-for="(att,i) in attachments" :key="i">
          <button class="btn-attachment" @click="$emit('open-fullscreen', att)">
            {{ att.name }}
          </button>
        </li>
      </ul>
      <button class="btn" @click="$emit('close')">Cerrar</button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  attachments: { type: Array, required: true },
  title: { type: String, required: true }
})
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top:0; left:0; width:100vw; height:100vh;
  background: rgba(0,0,0,0.7);
  display:flex; justify-content:center; align-items:center;
  z-index:1000;
}
.modal-content {
  background:#2a2a5a;
  padding:20px;
  border-radius:12px;
  width:90%;
  max-width:400px;
  color:white;
}
.modal-content h2 { margin-bottom:12px; }
.modal-content ul { list-style:none; padding:0; margin-bottom:12px; }
.modal-content li { margin-bottom:6px; }
.btn-attachment {
  background:#888; color:white;
  padding:6px 12px; border-radius:6px;
  border:none; cursor:pointer;
}
.btn-attachment:hover { background:#aaa; }
.btn {
  background:#444; color:white; padding:8px 16px; border-radius:6px; border:none; cursor:pointer;
}
.btn:hover { background:#666; }
</style>
