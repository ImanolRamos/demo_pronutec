<template>
  <div class="fullscreen-modal" @click.self="$emit('close')">
    <button class="btn-close" @click="$emit('close')">✖</button>
    <div class="fullscreen-content">
      <iframe v-if="attachment.type==='pdf'" :src="attachment.url" frameborder="0"></iframe>
      <video v-else-if="attachment.type==='video'" :src="attachment.url" controls autoplay></video>
      <img v-else-if="attachment.type==='image'" :src="attachment.url" />
      <a v-else :href="attachment.url" target="_blank">{{ attachment.name }}</a>
    </div>
  </div>
</template>

<script setup>
defineProps({
  attachment: { type: Object, required: true }
})
</script>

<style scoped>
.fullscreen-modal {
  position: fixed;
  top:0; left:0; width:100vw; height:100vh;
  background: rgba(0,0,0,0.95);
  display:flex; justify-content:center; align-items:center;
  z-index:2000; flex-direction: column;
}
.fullscreen-content {
  width:100%; height:100%;
  display:flex; justify-content:center; align-items:center;
}
.fullscreen-content iframe,
.fullscreen-content video,
.fullscreen-content img {
  width:100%; height:100%; object-fit: contain; border-radius:0;
}
.btn-close {
  position: fixed; top:16px; right:24px;
  font-size:32px; background:none; color:white;
  border:none; cursor:pointer; z-index:2100;
}
</style>
