<template>
  <div class="full-screen-container">

    <!-- Pantalla inicial -->
    <InitialScreen v-if="tasks.length === 0" 
                   @file-loaded="onFile" 
                   @use-sample="useSample" />

    <!-- Pantalla final -->
    <FinalScreen v-else-if="allDone" 
                 @restart="restart" 
                 @file-loaded="onFile" />

    <!-- Layout principal: lista lateral + paso activo -->
    <div v-else class="layout">
      <StepList :steps="visibleStepsData.steps" 
                :currentIndex="currentIndex" 
                :stepRefs="stepRefs" />

      <StepCard :current="current" 
                :currentIndex="currentIndex" 
                :tasks="tasks"
                :timeLeft="timeLeft"
                :paused="paused"
                :progressPercent="progressPercent"
                :doneThisStep="doneThisStep"
                @toggle-pause="togglePause"
                @complete-step="completeStep"
                @press-control="pressControl"
                @show-attachments="showAttachmentsModal = true" />
    </div>

    <!-- Modal de lista de adjuntos -->
    <AttachmentsModal v-if="showAttachmentsModal"
                      :attachments="current.attachments"
                      :title="current.title"
                      @close="showAttachmentsModal=false"
                      @open-fullscreen="openFullScreen" />

    <!-- Modal pantalla completa -->
    <FullScreenAttachment v-if="fullScreenAttachment"
                          :attachment="fullScreenAttachment"
                          @close="fullScreenAttachment=null" />
  </div>
</template>

<script setup>
import { ref, computed, onBeforeUnmount, watch, nextTick } from 'vue'

// Subcomponentes
import InitialScreen from './components/InitialScreen.vue'
import FinalScreen from './components/FinalScreen.vue'
import StepList from './components/StepList.vue'
import StepCard from './components/StepCard.vue'
import AttachmentsModal from './components/AttachmentsModal.vue'
import FullScreenAttachment from './components/FullScreenAttachment.vue'

// Estados principales
const tasks = ref([])
const currentIndex = ref(0)
const timeLeft = ref(0)
const paused = ref(false)
const intervalRef = ref(null)
const stepRefs = ref([])
const showAttachmentsModal = ref(false)
const fullScreenAttachment = ref(null)

// Computed
const current = computed(() => tasks.value[currentIndex.value] || {})
const progressPercent = computed(() => {
  if (!current.value || !current.value.duration) return 0
  return Math.round(((current.value.duration - timeLeft.value) / current.value.duration) * 100)
})
const doneThisStep = computed(() => current.value && current.value._done === true)
const isLast = computed(() => currentIndex.value >= tasks.value.length - 1)
const allDone = computed(() => tasks.value.length > 0 && currentIndex.value >= tasks.value.length)
const visibleStepsData = computed(() => ({ steps: tasks.value, start: 0 }))

// Timer
function startTimer(){
  stopTimer()
  if(!current.value || current.value.duration==null || doneThisStep.value) return
  timeLeft.value = Math.floor(current.value.duration)
  paused.value = false
  intervalRef.value = setInterval(()=>{
    if(paused.value) return
    if(!current.value.control){
      timeLeft.value = Math.max(0, timeLeft.value-1)
      if(timeLeft.value <= 0) markDoneAuto()
    }
  },1000)
}
function stopTimer(){ if(intervalRef.value){ clearInterval(intervalRef.value); intervalRef.value=null }}
function togglePause(){ paused.value = !paused.value }

// Acciones de pasos
function markDoneAuto(){ if(!current.value || current.value.control) return; current.value._done = true; advanceStep() }
function completeStep(){ if(!current.value || current.value.control) return; current.value._done = true; advanceStep() }
function pressControl(){ if(!current.value || !current.value.control) return; current.value._done = true; advanceStep() }

function advanceStep(){
  stopTimer()
  setTimeout(()=>{
    if(!isLast.value){ currentIndex.value+=1; startTimer() }
    else { currentIndex.value +=1 }
  },250)
}

// CSV
function parseCSV(text){
  const lines = text.split(/\r?\n/).map(l=>l.trim()).filter(Boolean)
  if(!lines.length) return []
  const header = lines.shift().split(',')
  return lines.map(line=>{
    const cols = line.split(',')
    const obj = {}
    header.forEach((h,i)=> obj[h]=cols[i]??'')
    return {
      id:Number(obj.id)||Date.now()+Math.floor(Math.random()*1000),
      title:obj.title||`Paso ${obj.id||''}`,
      description:obj.description||'',
      duration:Math.max(0,Number(obj.duration||0)),
      control:obj.control||'',
      attachments: [],
      _done:false
    }
  })
}

async function onFile(e){
  const f = e.target.files[0]; if(!f) return
  const text = await f.text()
  tasks.value = parseCSV(text)
  currentIndex.value=0
  if(tasks.value.length) startTimer()
}

function useSample(){
  const sample = `id,title,description,duration,control
1,Conectar manguera,Conectar la manguera al racor,20,
2,Encender válvula,Accionar la palanca hasta oír click,30,Pulsa "Válvula OK"
3,Medir presión,Anotar lectura del manómetro,15,
4,Confirmar seguridad,Comprobar que zona segura,45,Confirmar seguridad
5,Finalizar,Desconectar y cerrar,10,`
  tasks.value=parseCSV(sample)
  tasks.value[1].attachments = [
    { name:'Manual Válvula.pdf', url:'/attachments/manual_valvula.pdf', type:'pdf' },
    { name:'Video operación.mp4', url:'/attachments/video_operacion.mp4', type:'video' },
  ]
  tasks.value[2].attachments = [
    { name:'Diagrama.png', url:'/attachments/diagrama.png', type:'image' }
  ]
  currentIndex.value=0
  startTimer()
}

function restart() {
  tasks.value=[]; currentIndex.value=0; timeLeft.value=0; paused.value=false
}

function openFullScreen(att){
  fullScreenAttachment.value = att
  showAttachmentsModal.value=false
}

// Scroll automático
watch(currentIndex, async () => {
  await nextTick()
  const el = stepRefs.value[currentIndex.value]
  if(el) el.scrollIntoView({ behavior:'smooth', block:'center' })
})

onBeforeUnmount(()=>stopTimer())
</script>

<style scoped>
.full-screen-container { position: fixed; top:0; left:0; width:100vw; height:100vh; background:#1e1e1e; display:flex; justify-content:center; align-items:center; color:white; margin:0; padding:0; box-sizing:border-box; }
.layout { display:flex; gap:20px; width:90%; height:90%; align-items:stretch; }
</style>
