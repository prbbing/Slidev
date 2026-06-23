<!--
  JetDecayCones.vue
  Companion piece to JetTaggerInference.vue — same visual language (dark
  panel, IBM Plex type, same accent/track palette) but illustrates *why*
  b-jets look different: light-jet tracks all originate at the primary
  vertex (PV), while a b-jet has a b-hadron flying a short distance to a
  secondary vertex SV(b), decaying into tracks plus a charm hadron that
  flies further and decays again at SV(c).

  Sized for a single Slidev slide: the canvas itself is a square (so it
  reads cleanly as one figure rather than a wide banner). The component
  fills whatever box you give it (width:100%; height:100%) — control its
  actual size from the slide markdown by wrapping it in a sized container.
  See the two layout examples below.

  A) Full-bleed, title above, animation below at ~75% height:
    ---
    layout: full
    ---
    # Why b-jets look different
    <div style="height: 75%">
      <JetDecayCones />
    </div>

  B) Square animation with text beside it:
    ---
    layout: full
    ---
    <div class="flex h-full items-center gap-10 p-10">
      <div class="w-[46%] h-full">
        <JetDecayCones />
      </div>
      <div class="w-[54%]">
        ## Why b-jets look different
        - Light-jet tracks all trace back to the primary vertex
        - A b-hadron flies a short distance before decaying at SV(b)
        - One decay product is a charm hadron, which flies on and
          decays again at SV(c)
      </div>
    </div>
-->
<template>
  <div class="jet-cones">
    <div class="stage">
      <svg viewBox="0 0 660 700" xmlns="http://www.w3.org/2000/svg">

        <!-- ============== LIGHT JET ============== -->
        <g class="cone-outline cone-light">
          <ellipse cx="190" cy="140" rx="85" ry="20"/>
          <line x1="105" y1="140" x2="190" y2="620"/>
          <line x1="275" y1="140" x2="190" y2="620"/>
        </g>
        <text x="190" y="94" text-anchor="middle" class="panel-title">Light jet</text>

        <circle class="dot-pv" cx="190" cy="620" r="8"/>
        <text x="190" y="656" text-anchor="middle" class="dim-label" font-size="12">PV</text>

        <g v-for="(t,i) in lightTracks" :key="'lt'+i">
          <line :x1="t.x1" :y1="t.y1" :x2="t.x2" :y2="t.y2"
                class="track track-prompt" :style="trackStyle(t, revealed(1), i)"/>
          <polygon :points="arrowHead(t.x1,t.y1,t.x2,t.y2)" class="arrowhead arrow-prompt"
                   :style="{ opacity: revealed(1) ? 1 : 0, transitionDelay: (0.35+i*0.08)+'s' }"/>
        </g>

        <!-- ============== B-JET ============== -->
        <g class="cone-outline cone-b">
          <ellipse cx="490" cy="140" rx="85" ry="20"/>
          <line x1="405" y1="140" x2="490" y2="620"/>
          <line x1="575" y1="140" x2="490" y2="620"/>
        </g>
        <text x="490" y="94" text-anchor="middle" class="panel-title">b-jet</text>

        <circle class="dot-pv" cx="490" cy="620" r="8"/>
        <text x="490" y="656" text-anchor="middle" class="dim-label" font-size="12">PV</text>

        <g v-for="(t,i) in bPromptTracks" :key="'bp'+i">
          <line :x1="t.x1" :y1="t.y1" :x2="t.x2" :y2="t.y2"
                class="track track-prompt" :style="trackStyle(t, revealed(1), i)"/>
          <polygon :points="arrowHead(t.x1,t.y1,t.x2,t.y2)" class="arrowhead arrow-prompt"
                   :style="{ opacity: revealed(1) ? 1 : 0, transitionDelay: (0.35+i*0.08)+'s' }"/>
        </g>

        <!-- b-hadron flight: PV -> SV(b) -->
        <line :x1="flight1.x1" :y1="flight1.y1" :x2="flight1.x2" :y2="flight1.y2"
              class="flight-line" :style="{ opacity: revealed(2) ? 1 : 0 }"/>

        <circle v-show="revealed(2)" class="dot-svb" :cx="svB.x" :cy="svB.y" r="8"/>
        <text v-show="revealed(2)" :x="svB.x+20" :y="svB.y+6" class="dim-label" font-size="12">SV(b)</text>
        <circle v-if="flashB" class="decay-flash flash-svb" :cx="svB.x" :cy="svB.y" r="7"/>

        <g v-for="(t,i) in svBTracks" :key="'sb'+i">
          <line :x1="t.x1" :y1="t.y1" :x2="t.x2" :y2="t.y2"
                class="track track-svb" :style="trackStyle(t, revealed(3), i)"/>
          <polygon :points="arrowHead(t.x1,t.y1,t.x2,t.y2)" class="arrowhead arrow-svb"
                   :style="{ opacity: revealed(3) ? 1 : 0, transitionDelay: (0.35+i*0.08)+'s' }"/>
        </g>

        <!-- charm-hadron flight: SV(b) -> SV(c) -->
        <line :x1="flight2.x1" :y1="flight2.y1" :x2="flight2.x2" :y2="flight2.y2"
              class="flight-line" :style="{ opacity: revealed(3) ? 1 : 0 }"/>

        <circle v-show="revealed(4)" class="dot-svc" :cx="svC.x" :cy="svC.y" r="8"/>
        <text v-show="revealed(4)" :x="svC.x+20" :y="svC.y+6" class="dim-label" font-size="12">SV(c)</text>
        <circle v-if="flashC" class="decay-flash flash-svc" :cx="svC.x" :cy="svC.y" r="7"/>

        <g v-for="(t,i) in svCTracks" :key="'sc'+i">
          <line :x1="t.x1" :y1="t.y1" :x2="t.x2" :y2="t.y2"
                class="track track-svc" :style="trackStyle(t, revealed(4), i)"/>
          <polygon :points="arrowHead(t.x1,t.y1,t.x2,t.y2)" class="arrowhead arrow-svc"
                   :style="{ opacity: revealed(4) ? 1 : 0, transitionDelay: (0.35+i*0.08)+'s' }"/>
        </g>

      </svg>
    </div>

    <div class="controls">
      <button class="btn play" @click="togglePlay" :aria-label="playing ? 'Pause' : 'Play'">
        <span>{{ playing ? '❙❙ Pause' : '▶ Play' }}</span>
      </button>
      <button class="btn" @click="step(-1)" aria-label="Previous step">◂ Prev</button>
      <button class="btn" @click="step(1)" aria-label="Next step">Next ▸</button>
      <button class="btn" @click="reset" aria-label="Restart">↺ Restart</button>

      <div class="dots" role="tablist" aria-label="Decay stages">
        <button v-for="(s,i) in stepLabels" :key="'dot'+i" class="dot"
                :class="{ 'is-active': current===i, 'is-done': current>i }"
                @click="goTo(i)" :aria-label="s" :aria-current="current===i"></button>
      </div>

      <div class="counter">STEP <b>{{ current+1 }}</b> / {{ stepLabels.length }}</div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'

const dist = (x1,y1,x2,y2) => Math.hypot(x2-x1, y2-y1)

function arrowHead(x1,y1,x2,y2,size=12){
  const ang = Math.atan2(y2-y1, x2-x1)
  const a1 = ang + Math.PI*0.85
  const a2 = ang - Math.PI*0.85
  const p1x = (x2 + size*Math.cos(a1)).toFixed(1), p1y = (y2 + size*Math.sin(a1)).toFixed(1)
  const p2x = (x2 + size*Math.cos(a2)).toFixed(1), p2y = (y2 + size*Math.sin(a2)).toFixed(1)
  return `${x2},${y2} ${p1x},${p1y} ${p2x},${p2y}`
}

function trackStyle(t, show, i){
  return {
    strokeDasharray: t.len,
    strokeDashoffset: show ? 0 : t.len,
    transitionDelay: (i*0.1)+'s'
  }
}

// ---- geometry (square-ish canvas: 660 x 700) ----
const lightApex = { x:190, y:620 }
const lightRimY = 140
const lightTracks = [115,150,190,228,262].map(x => ({
  x1: lightApex.x, y1: lightApex.y, x2: x, y2: lightRimY, len: dist(lightApex.x,lightApex.y,x,lightRimY)
}))

const bApex = { x:490, y:620 }
const bRimY = 140
const bPromptTracks = [425,490,560].map(x => ({
  x1: bApex.x, y1: bApex.y, x2: x, y2: bRimY, len: dist(bApex.x,bApex.y,x,bRimY)
}))

const svB = { x:490, y:540 }
const svC = { x:490, y:440 }

const svBTracks = [415,485,565].map(x => ({
  x1: svB.x, y1: svB.y, x2: x, y2: bRimY, len: dist(svB.x,svB.y,x,bRimY)
}))
const svCTracks = [450,530].map(x => ({
  x1: svC.x, y1: svC.y, x2: x, y2: bRimY, len: dist(svC.x,svC.y,x,bRimY)
}))

const flight1 = { x1: bApex.x, y1: bApex.y, x2: svB.x, y2: svB.y }
const flight2 = { x1: svB.x,   y1: svB.y,   x2: svC.x, y2: svC.y }

// ---- step / reveal state ----
const stepLabels = [
  'Primary vertex',
  'Prompt tracks',
  'b-hadron flies to SV(b)',
  'b-hadron decays + charm flies on',
  'Charm decays at SV(c)',
]
const current = ref(0)
const playing = ref(true)
let timer = null
const INTERVAL = 1700

function revealed(threshold){ return current.value >= threshold }

function advance(){ current.value = (current.value + 1) % stepLabels.length }
function startTimer(){ stopTimer(); timer = setInterval(advance, INTERVAL) }
function stopTimer(){ if (timer) { clearInterval(timer); timer = null } }
function togglePlay(){ playing.value = !playing.value; playing.value ? startTimer() : stopTimer() }
function goTo(i){ current.value = i; playing.value = false; stopTimer() }
function step(dir){
  current.value = (current.value + dir + stepLabels.length) % stepLabels.length
  playing.value = false
  stopTimer()
}
function reset(){ current.value = 0; playing.value = true; startTimer() }

// one-shot "decay flash" rings, triggered when a vertex first appears
const flashB = ref(false)
const flashC = ref(false)
watch(current, (val, old) => {
  if (val >= 2 && old < 2) { flashB.value = true; setTimeout(() => flashB.value = false, 650) }
  if (val >= 4 && old < 4) { flashC.value = true; setTimeout(() => flashC.value = false, 650) }
})

onMounted(startTimer)
onBeforeUnmount(stopTimer)
</script>

<style scoped>
.jet-cones{
  --bg: #0a0d12;
  --panel: #11161d;
  --border: #232b37;
  --text: #e7eaf0;
  --text-dim: #828d9c;
  --text-faint: #4b5563;
  --accent: #5eead4;
  --lavender: #93a5c9;
  --rose: #e0507a;
  --orange: #f5a524;
  --red: #f0615f;

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
.jet-cones *{ box-sizing: border-box; }

.stage{
  flex: 1 1 auto;
  min-height: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--panel);
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 10px;
  overflow: hidden;
}
.stage svg{ display:block; width:100%; height:100%; }

/* halo behind every label so it stays legible over lines/cones */
text{
  paint-order: stroke;
  stroke: var(--panel);
  stroke-width: 5px;
  stroke-linejoin: round;
}

.panel-title{ font-size: 17px; font-weight: 600; fill: var(--text); }
.dim-label{
  font-family:'IBM Plex Mono', monospace;
  fill: var(--text-dim);
  letter-spacing: 0.02em;
}

.cone-outline{ fill:none; stroke-width: 3.5; stroke-linecap: round; }
.cone-light{ stroke: rgba(94,234,212,0.55); }
.cone-b{ stroke: rgba(224,80,122,0.5); }

.track{ fill:none; stroke-width: 4; stroke-linecap: round; transition: stroke-dashoffset .55s ease; }
.track-prompt{ stroke: var(--accent); }
.track-svb{ stroke: var(--rose); }
.track-svc{ stroke: var(--orange); }

.arrowhead{ transition: opacity .3s ease; }
.arrow-prompt{ fill: var(--accent); }
.arrow-svb{ fill: var(--rose); }
.arrow-svc{ fill: var(--orange); }

.flight-line{ stroke: var(--lavender); stroke-width: 3.5; stroke-dasharray: 10 7; fill:none; transition: opacity .4s ease; }

.dot-pv{ fill: var(--red); stroke: #0a0d12; stroke-width: 2.2; }
.dot-svb{ fill: var(--rose); stroke: #0a0d12; stroke-width: 2.2; transition: opacity .3s ease; }
.dot-svc{ fill: var(--orange); stroke: #0a0d12; stroke-width: 2.2; transition: opacity .3s ease; }

.decay-flash{
  fill: none;
  stroke: var(--text);
  stroke-width: 3;
  pointer-events: none;
  animation: decayFlash .6s ease-out forwards;
}
@keyframes decayFlash{
  from{ r: 7; opacity: 0.9; stroke-width: 4.5; }
  to{ r: 24; opacity: 0; stroke-width: 1; }
}

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

.dots{ display:flex; gap:7px; align-items:center; }
.dot{
  width: 9px; height: 9px; border-radius: 50%;
  background: var(--text-faint);
  cursor: pointer;
  border: none;
  padding: 0;
  transition: background .2s, transform .2s;
}
.dot:hover{ transform: scale(1.25); }
.dot.is-done{ background: #5b6573; }
.dot.is-active{ background: var(--accent); transform: scale(1.35); box-shadow: 0 0 8px var(--accent); }

.counter{
  font-family:'IBM Plex Mono', monospace;
  font-size: 11.5px;
  color: var(--text-dim);
  margin-left: auto;
}
.counter b{ color: var(--accent); }

@media (prefers-reduced-motion: reduce){
  .decay-flash{ animation: none !important; }
}
</style>
