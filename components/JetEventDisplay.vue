<!--
  JetEventDisplay.vue
  Third companion piece to JetTaggerInference.vue / JetDecayCones.vue — same
  dark visual language, but this one is a dense "event display": many
  vertices spread along a line, each firing a spray of randomly colored
  tracks in every direction, echoing a real collision-event visualization
  with lots of pileup vertices. Tracks pulse in (grow outward, staggered),
  hold, fade out, and repeat. Deterministic (seeded RNG) so it looks the
  same every time you open the slide.

  Usage (e.g. in Slidev):
    ---
    layout: full
    ---
    <JetEventDisplay />

  Fills whatever box it's given (width:100%; height:100%) — wrap it in a
  sized container from your slide markdown if you want it smaller, e.g.
    <div style="height: 80%"><JetEventDisplay /></div>
-->
<template>
  <div class="event-display">
    <div class="stage">
      <svg viewBox="0 0 1600 560" xmlns="http://www.w3.org/2000/svg">
        <g class="tracks" :class="{ 'is-visible': visible }">
          <line v-for="(t,i) in tracks" :key="'tr'+i"
                :x1="t.x1" :y1="t.y1" :x2="t.x2" :y2="t.y2"
                class="track" :class="{ growing: fired }"
                :style="{
                  stroke: t.color,
                  strokeWidth: t.width,
                  opacity: t.op,
                  strokeDasharray: t.len,
                  strokeDashoffset: fired ? 0 : t.len,
                  transitionDelay: t.delay + 's'
                }"/>

          <line :x1="flight1.x1" :y1="flight1.y1" :x2="flight1.x2" :y2="flight1.y2" class="flight-line"/>
          <line :x1="flight2.x1" :y1="flight2.y1" :x2="flight2.x2" :y2="flight2.y2" class="flight-line"/>

          <line v-for="(t,i) in decayTracks" :key="'dt'+i"
                :x1="t.x1" :y1="t.y1" :x2="t.x2" :y2="t.y2"
                class="track" :class="{ growing: fired }"
                :style="{
                  stroke: t.color,
                  strokeWidth: t.width,
                  opacity: t.op,
                  strokeDasharray: t.len,
                  strokeDashoffset: fired ? 0 : t.len,
                  transitionDelay: t.delay + 's'
                }"/>

          <circle :cx="svB.x" :cy="svB.y" :r="svB.r" class="sub-vertex" :style="{ fill: svB.color }"/>
          <circle :cx="svC.x" :cy="svC.y" :r="svC.r" class="sub-vertex" :style="{ fill: svC.color }"/>
        </g>
        <g class="vertices" :class="{ 'is-visible': visible }">
          <circle v-for="(v,i) in vertices" :key="'v'+i"
                  :cx="v.x" :cy="v.y" :r="v.r" class="vertex"
                  :style="{ fill: v.color }"/>
        </g>
      </svg>
    </div>

    <div class="controls">
      <button class="btn play" @click="togglePlay" :aria-label="playing ? 'Pause' : 'Play'">
        <span>{{ playing ? '❙❙ Pause' : '▶ Play' }}</span>
      </button>
      <button class="btn" @click="replayNow" aria-label="Replay burst">⟲ Replay burst</button>
      <div class="counter">{{ vertices.length }} vertices · {{ tracks.length }} tracks</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const dist = (x1,y1,x2,y2) => Math.hypot(x2-x1, y2-y1)

// seeded RNG so the layout is stable across reloads
function mulberry32(seed){
  return function(){
    seed |= 0; seed = (seed + 0x6D2B79F5) | 0
    let t = Math.imul(seed ^ (seed >>> 15), 1 | seed)
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296
  }
}
const rand = mulberry32(20260623)
const pick = (arr) => arr[Math.floor(rand() * arr.length)]

const palette = ['#f0615f','#f5c542','#e0507a','#34d399','#5eead4','#f5a524','#93a5c9','#4d8cf6']

// ---- generate vertices along a rough horizontal line ----
const NUM_V = 13
const VERTEX_R = 9
const LINE_Y = 280
const vertices = []
for (let i = 0; i < NUM_V; i++) {
  const baseX = 60 + i * ((1600 - 120) / (NUM_V - 1))
  vertices.push({
    x: baseX + (rand() - 0.5) * 24,
    y: LINE_Y + (rand() - 0.5) * 14,
    r: VERTEX_R,
    color: pick(palette),
  })
}

// pick a couple of vertices to host one bold "highlight" track each
const highlightSet = new Set()
while (highlightSet.size < 2) highlightSet.add(Math.floor(rand() * NUM_V))

// ---- generate a spray of tracks per vertex ----
const tracks = []
vertices.forEach((v, vi) => {
  const n = 18 + Math.floor(rand() * 13) // 18..30 tracks per vertex
  for (let k = 0; k < n; k++) {
    const isHi = highlightSet.has(vi) && k === 0
    const angle = rand() * Math.PI * 2
    const len = isHi ? 1300 + rand() * 400 : 280 + rand() * 1100
    const x2 = v.x + Math.cos(angle) * len
    const y2 = v.y + Math.sin(angle) * len
    tracks.push({
      x1: v.x, y1: v.y, x2, y2,
      len: dist(v.x, v.y, x2, y2),
      color: isHi ? '#f5c542' : pick(palette),
      width: isHi ? 4.2 : 1 + rand() * 0.6,
      op: isHi ? 1 : 0.45 + rand() * 0.3,
      delay: rand() * 1.1 + vi * 0.05,
    })
  }
})

// ---- one vertex is a b-hadron decay: PV -> SV(b) -> SV(c) ----
const hostIdx = Math.floor(NUM_V / 2)
const pv = vertices[hostIdx]
const svB = { x: pv.x + 38, y: pv.y - 24, r: 5.5, color: '#e0507a' }
const svC = { x: svB.x + 36, y: svB.y - 22, r: 5, color: '#f5a524' }

const flight1 = { x1: pv.x, y1: pv.y, x2: svB.x, y2: svB.y }
const flight2 = { x1: svB.x, y1: svB.y, x2: svC.x, y2: svC.y }

const decayTracks = []
function addDecayTracks(origin, baseAngle, count, color, spread){
  for (let k = 0; k < count; k++) {
    const angle = baseAngle + (rand() - 0.5) * spread // fan loosely around the parent's flight direction
    const len = 260 + rand() * 700
    const x2 = origin.x + Math.cos(angle) * len
    const y2 = origin.y + Math.sin(angle) * len
    decayTracks.push({
      x1: origin.x, y1: origin.y, x2, y2,
      len: dist(origin.x, origin.y, x2, y2),
      color, width: 1.8 + rand() * 0.5, op: 0.85 + rand() * 0.15,
      delay: rand() * 0.6 + 1.1,
    })
  }
}
const flightAngle1 = Math.atan2(svB.y - pv.y, svB.x - pv.x)
const flightAngle2 = Math.atan2(svC.y - svB.y, svC.x - svB.x)
addDecayTracks(svB, flightAngle1, 4, '#e0507a', Math.PI * 0.55)
addDecayTracks(svC, flightAngle2, 3, '#f5a524', Math.PI * 0.45)

// ---- burst / hold / fade / repeat cycle ----
const playing = ref(true)
const visible = ref(false)
const fired = ref(false)
let gen = 0

function sleep(ms){ return new Promise(r => setTimeout(r, ms)) }

async function cycle(){
  const myGen = ++gen
  while (playing.value && myGen === gen) {
    visible.value = false
    fired.value = false
    await sleep(300); if (myGen !== gen || !playing.value) return
    visible.value = true
    await sleep(80); if (myGen !== gen || !playing.value) return
    fired.value = true
    await sleep(4200); if (myGen !== gen || !playing.value) return
    visible.value = false
    await sleep(650); if (myGen !== gen || !playing.value) return
  }
}

function togglePlay(){
  playing.value = !playing.value
  if (playing.value) cycle()
}
function replayNow(){
  playing.value = true
  cycle()
}

onMounted(cycle)
onBeforeUnmount(() => { playing.value = false; gen++ })
</script>

<style scoped>
.event-display{
  --bg: #0a0d12;
  --panel: #11161d;
  --border: #232b37;
  --text: #e7eaf0;
  --text-dim: #828d9c;
  --text-faint: #4b5563;
  --accent: #5eead4;

  background: var(--bg);
  color: var(--text);
  font-family: 'IBM Plex Sans', sans-serif;
  -webkit-font-smoothing: antialiased;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 18px 28px;
}
.event-display *{ box-sizing: border-box; }

.stage{
  flex: 1 1 auto;
  min-height: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #000;
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 6px;
  overflow: hidden;
}
.stage svg{ display:block; width:100%; height:100%; }

.tracks{ opacity: 0; transition: opacity .6s ease; }
.tracks.is-visible{ opacity: 1; }

.vertices{ opacity: 0; transition: opacity .6s ease; }
.vertices.is-visible{ opacity: 1; }

.track{ fill:none; }
.track.growing{ transition: stroke-dashoffset .65s cubic-bezier(.22,.8,.25,1); }

.flight-line{ stroke: #93a5c9; stroke-width: 2.4; stroke-dasharray: 8 6; fill:none; opacity: 0.9; }

.vertex{ stroke: #000; stroke-width: 1.5; }
.sub-vertex{ stroke: #000; stroke-width: 1.2; }

.controls{
  flex: 0 0 auto;
  display: flex;
  align-items: center;
  gap: 14px;
  flex-wrap: wrap;
}
.btn{
  font-family:'IBM Plex Mono', monospace;
  font-size: 12px;
  letter-spacing: 0.04em;
  background: #161c25;
  border: 1px solid var(--border);
  color: var(--text);
  border-radius: 8px;
  padding: 8px 14px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 7px;
  transition: border-color .2s, color .2s;
}
.btn:hover{ border-color: var(--accent); color: var(--accent); }
.btn:focus-visible{ outline: 2px solid var(--accent); outline-offset: 2px; }
.btn.play{ background: var(--accent); color: #06231e; border-color: var(--accent); font-weight:600; }
.btn.play:hover{ color:#06231e; }

.counter{
  font-family:'IBM Plex Mono', monospace;
  font-size: 11.5px;
  color: var(--text-dim);
  margin-left: auto;
}

@media (prefers-reduced-motion: reduce){
  .track.growing{ transition: none !important; }
  .tracks{ transition: none !important; }
}
</style>
