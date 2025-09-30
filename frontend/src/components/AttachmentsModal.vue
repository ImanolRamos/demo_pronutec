<template>
  <div class="attachments-modal">
    <h3>{{ title }}</h3>
    <div v-for="att in attachments" :key="att.url" class="attachment-item">
      
      <!-- PDF o cualquier enlace -->
      <a v-if="att.type==='pdf'" :href="att.url" target="_blank">{{ att.name }}</a>
      
      <!-- Imagen -->
      <img v-else-if="att.type==='image'" :src="att.url" :alt="att.name" class="attachment-img" />

      <!-- Video -->
      <video v-else-if="att.type==='video'" controls class="attachment-video">
        <source :src="att.url" type="video/mp4">
        Tu navegador no soporta videos.
      </video>

      <!-- Link por defecto -->
      <a v-else :href="att.url" target="_blank">{{ att.name }}</a>

    </div>
    <button @click="$emit('close')">Cerrar</button>
  </div>
</template>

<script setup>
defineProps({
  attachments: Array,
  title: String
})
defineEmits(['close'])
</script>

<style scoped>
.attachments-modal { background:#222; padding:20px; border-radius:12px; max-width:600px; width:90%; color:white; }
.attachment-item { margin-bottom:16px; }
.attachment-img { max-width:100%; border-radius:8px; }
.attachment-video { max-width:100%; border-radius:8px; }
button { margin-top:12px; background:#28a745; border:none; padding:8px 16px; color:white; border-radius:8px; cursor:pointer; }
</style>
