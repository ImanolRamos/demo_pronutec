<template>
  <div class="full-screen-container">
    <!-- Pantalla inicial -->
    <div v-if="tasks.length === 0" class="initial-screen">
      <h1>Guía Profesional de Operaciones</h1>
      <p>Antes de comenzar, cargue el archivo CSV con los pasos que debe seguir el operario.</p>
      <div class="btn-group">
        <label class="btn" for="file">Cargar CSV</label>
        <input id="file" ref="fileInput" type="file" accept=".csv,text/csv" @change="onFile" style="display:none" />
        <button class="btn" @click="useSample">Usar CSV de ejemplo</button>
      </div>
    </div>

    <!-- Pantalla final -->
    <div v-else-if="allDone" class="final-screen">
      <h1>🎉 Todas las tareas completadas 🎉</h1>
      <p>¡Buen trabajo! Puede revisar los pasos completados o cargar otro CSV para empezar de nuevo.</p>
      <div class="btn-group">
        <button class="btn" @click="restart">Reiniciar guía</button>
        <label class="btn" for="file">Cargar otro CSV</label>
        <input id="file" ref="fileInput" type="file" accept=".csv,text/csv" @change="onFile" style="display:none" />
      </div>
    </div>

    <!-- Card central con lista lateral y paso activo -->
    <div v-else class="layout">
      <!-- Lista lateral -->
      <div class="steps-panel">
        <h3>Pasos</h3>
        <div class="step-list">
          <div v-for="(t,i) in visibleStepsData.steps" :key="t.id"
               :class="['step-item', {active: i + visibleStepsData.start === currentIndex, done: (i + visibleStepsData.start) < currentIndex}]"
               :ref="el => stepRefs[i + visibleStepsData.start] = el">
            <div class="step-number">{{ i + visibleStepsData.start + 1 }}</div>
            <div class="step-title-small">{{ t.title }}</div>
            <div class="step-duration-small">
              {{ t.duration }}s 
              <span v-if="t.control">· {{ t.control }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Paso activo -->
      <div class="step-card">
        <div class="step-header">
          <div class="step-counter">Paso {{ currentIndex+1 }} de {{ tasks.length }}</div>
          <h1 class="step-title">{{ current.title }}</h1>
          <p class="step-description">{{ current.description }}</p>
          <p v-if="current.control" class="blocking-warning">⚠ Este paso requiere control: "{{ current.control }}"</p>
        </div>

        <!-- Temporizador -->
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
          <button class="btn-pause" @click="togglePause">{{ paused ? 'Reanudar' : 'Pausar' }}</button>
        </div>

        <!-- Botones de acción -->
        <div class="action-container">
          <button v-if="current.control" class="btn-blocking" @click="pressControl" :disabled="doneThisStep">
            {{ current.control }}
          </button>
          <button v-else class="btn-nonblocking" @click="completeStep" :disabled="doneThisStep">
            Marcar completado
          </button>

          <!-- Botón de adjuntos -->
          <button v-if="current.attachments && current.attachments.length"
                  class="btn-attachments"
                  @click="showAttachmentsModal = true">
            Ver adjuntos ({{ current.attachments.length }})
          </button>
        </div>

        <!-- Estado -->
        <div class="step-status">Estado: <strong>{{ doneThisStep ? 'Completado' : (paused ? 'Pausado' : 'En curso') }}</strong></div>
      </div>
    </div>

    <!-- Modal de lista de adjuntos -->
    <div v-if="showAttachmentsModal" class="modal-overlay" @click.self="showAttachmentsModal=false">
      <div class="modal-content">
        <h2>Adjuntos de "{{ current.title }}"</h2>
        <ul>
          <li v-for="(att,i) in current.attachments" :key="i">
            <button class="btn-attachment" @click="openFullScreen(att)">{{ att.name }}</button>
          </li>
        </ul>
        <button class="btn" @click="showAttachmentsModal=false">Cerrar</button>
      </div>
    </div>

    <!-- Modal pantalla completa para previsualización -->
    <div v-if="fullScreenAttachment" class="fullscreen-modal" @click.self="fullScreenAttachment=null">
      <button class="btn-close" @click="fullScreenAttachment=null">✖</button>
      <div class="fullscreen-content">
        <iframe v-if="fullScreenAttachment.type==='pdf'" :src="fullScreenAttachment.url" frameborder="0"></iframe>
        <video v-else-if="fullScreenAttachment.type==='video'" :src="fullScreenAttachment.url" controls autoplay></video>
        <img v-else-if="fullScreenAttachment.type==='image'" :src="fullScreenAttachment.url" />
        <a v-else :href="fullScreenAttachment.url" target="_blank">{{ fullScreenAttachment.name }}</a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onBeforeUnmount, watch, nextTick } from 'vue'

const tasks = ref([])
const currentIndex = ref(0)
const timeLeft = ref(0)
const paused = ref(false)
const intervalRef = ref(null)
const stepRefs = ref([])
const showAttachmentsModal = ref(false)
const fullScreenAttachment = ref(null)

const current = computed(() => tasks.value[currentIndex.value] || {})
const progressPercent = computed(() => {
  if (!current.value || !current.value.duration) return 0
  return Math.round(((current.value.duration - timeLeft.value) / current.value.duration) * 100)
})
const doneThisStep = computed(() => current.value && current.value._done === true)
const isLast = computed(() => currentIndex.value >= tasks.value.length - 1)
const allDone = computed(() => tasks.value.length > 0 && currentIndex.value >= tasks.value.length)
const visibleStepsData = computed(() => ({ steps: tasks.value, start: 0 }))

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

  // Ejemplo de adjuntos
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

// Scroll automático a paso activo
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
.steps-panel { width:250px; background:#2a2a2a; border-radius:10px; padding:12px; display:flex; flex-direction:column; }
.step-list { display:flex; flex-direction:column; gap:6px; flex:1; overflow:hidden; }
.step-item { padding:6px 8px; border-radius:6px; background:#3a3a3a; color:#ccc; display:flex; flex-direction:column; }
.step-item.active { background:#4444aa; color:white; font-weight:bold; box-shadow:0 0 8px #888; }
.step-item.done { background:#555; color:#999; }
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
.initial-screen, .final-screen { text-align:center; }
.final-screen h1 { font-size:32px; color:#ffd700; margin-bottom:12px; }
.final-screen p { font-size:18px; margin-bottom:20px; }
.btn-group { display:flex; gap:12px; justify-content:center; margin-top:20px; }
.btn { background:#444; color:white; padding:12px 20px; border-radius:8px; cursor:pointer; border:none; transition:all 0.2s; }
.btn:hover { background:#666; }
.modal-overlay { position: fixed; top:0; left:0; width:100vw; height:100vh; background: rgba(0,0,0,0.7); display:flex; justify-content:center; align-items:center; z-index:1000; }
.modal-content { background:#2a2a5a; padding:20px; border-radius:12px; width:90%; max-width:400px; color:white; }
.modal-content h2 { margin-bottom:12px; }
.modal-content ul { list-style:none; padding:0; margin-bottom:12px; }
.modal-content li { margin-bottom:6px; }
.fullscreen-modal { position: fixed; top:0; left:0; width:100vw; height:100vh; background: rgba(0,0,0,0.95); display:flex; justify-content:center; align-items:center; z-index:2000; flex-direction: column; }
.fullscreen-content { width:100%; height:100%; display:flex; justify-content:center; align-items:center; }
.fullscreen-content iframe, .fullscreen-content video, .fullscreen-content img { width:100%; height:100%; object-fit: contain; border-radius:0; }
.btn-close { position: fixed; top:16px; right:24px; font-size:32px; background:none; color:white; border:none; cursor:pointer; z-index:2100; }
.btn-attachment { background:#888; color:white; padding:6px 12px; border-radius:6px; border:none; cursor:pointer; }
</style>
