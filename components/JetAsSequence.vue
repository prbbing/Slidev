<!--
  JetAsSequence.vue
  Fifth companion piece (alongside JetTaggerInference.vue, JetDecayCones.vue,
  JetEventDisplay.vue, JetTaggerEvolution.vue). Reuses the same b-jet cone
  (PV -> SV(b) -> SV(c)) from JetDecayCones.vue, but instead of stopping at
  the cone, each track is "pulled out" of the cone and laid down as one
  token in a flat sequence on the right — the visual argument for why a
  jet can be fed to a sequence model (RNN/Transformer/etc.) the same way a
  sentence's words become tokens.

  Usage (e.g. in Slidev):
    ---
    layout: full
    ---
    <JetAsSequence />

  Fills whatever box it's given (width:100%; height:100%).
-->
<template>
  <div class="jet-sequence">
    <div class="stage">
      <svg viewBox="0 0 1900 760" xmlns="http://www.w3.org/2000/svg">

        <text x="280" y="90" text-anchor="middle" class="panel-title">Jet</text>
        <text :x="seqCenterX" y="300" text-anchor="middle" class="panel-title">Sequence</text>

        <!-- ===================== CONE ===================== -->
        <g class="cone-outline">
          <ellipse cx="280" cy="150" rx="120" ry="28"/>
          <line x1="160" y1="150" x2="280" y2="640"/>
          <line x1="400" y1="150" x2="280" y2="640"/>
        </g>
        <circle class="dot-pv" cx="280" cy="640" r="8"/>

        <line :x1="flight1.x1" :y1="flight1.y1" :x2="flight1.x2" :y2="flight1.y2"
              class="flight-line" :style="{ opacity: revealed(2) ? 1 : 0 }"/>
        <line :x1="flight2.x1" :y1="flight2.y1" :x2="flight2.x2" :y2="flight2.y2"
              class="flight-line" :style="{ opacity: revealed(3) ? 1 : 0 }"/>
        <circle v-show="revealed(2)" class="dot-svb" :cx="svB.x" :cy="svB.y" r="8"/>
        <circle v-show="revealed(3)" class="dot-svc" :cx="svC.x" :cy="svC.y" r="7"/>

        <g v-for="(t,i) in allTracks" :key="'trk'+i">
          <line :x1="t.x1" :y1="t.y1" :x2="t.x2" :y2="t.y2"
                class="track"
                :style="{
                  stroke: t.color,
                  strokeDasharray: t.len,
                  strokeDashoffset: revealed(t.group) ? 0 : t.len
                }"/>
          <polygon :points="arrowHead(t.x1,t.y1,t.x2,t.y2)" class="arrowhead"
                   :style="{ fill: t.color, opacity: revealed(t.group) ? 1 : 0 }"/>
        </g>

        <!-- ===================== CONNECTORS ===================== -->
        <path v-for="(c,i) in connectors" :key="'conn'+i" :d="c.d"
              class="connector"
              :style="{
                stroke: c.color,
                strokeDasharray: c.len,
                strokeDashoffset: revealed(c.group) ? 0 : c.len,
                transitionDelay: '.15s'
              }"/>

        <!-- ===================== TOKEN SEQUENCE ===================== -->
        <g v-for="(tok,i) in tokens" :key="'tok'+i" class="token"
           :style="{ opacity: revealed(tok.group) ? 1 : 0, transitionDelay: (revealed(tok.group) ? 0.35 : 0)+'s' }">
          <rect :x="tok.boxX" :y="tok.boxY" width="100" height="10" :style="{ fill: tok.color }"/>
          <rect :x="tok.boxX" :y="tok.boxY+10" width="100" height="90" rx="6" class="token-body"/>
          <text :x="tok.boxX+50" :y="tok.boxY+62" text-anchor="middle" class="token-label">{{ tok.label }}</text>
        </g>

        <text :x="seqCenterX" :y="tokenRowBottom+34" text-anchor="middle" class="dim-label"
              :style="{ opacity: revealed(3) ? 1 : 0 }">input sequence</text>

        <!-- ===================== STEP 4: TOKEN MATRIX ===================== -->
        <g v-for="(tok,i) in tokens" :key="'col'+i">
          <rect v-for="cell in tok.cells" :key="cell.j"
                :x="tok.boxX" :y="cell.y" width="100" :height="cellH" rx="3"
                class="matrix-cell"
                :style="{ fill: tok.color, opacity: revealed(4) ? cell.opacity : 0, transitionDelay: cell.delay+'s' }"/>
        </g>

        <path :d="leftBracket" class="bracket" :style="{ opacity: revealed(4) ? 1 : 0 }"/>
        <path :d="rightBracket" class="bracket" :style="{ opacity: revealed(4) ? 1 : 0 }"/>

        <text :x="seqCenterX" :y="matrixEndY+34" text-anchor="middle" class="dim-label"
              :style="{ opacity: revealed(4) ? 1 : 0 }">embedding matrix — one column per token</text>

      </svg>
    </div>

    <div class="controls">
      <button class="btn play" @click="togglePlay" :aria-label="playing ? 'Pause' : 'Play'">
        <span>{{ playing ? '❙❙ Pause' : '▶ Play' }}</span>
      </button>
      <button class="btn" @click="step(-1)" aria-label="Previous step">◂ Prev</button>
      <button class="btn" @click="step(1)" aria-label="Next step">Next ▸</button>
      <button class="btn" @click="reset" aria-label="Restart">↺ Restart</button>

      <div class="dots" role="tablist" aria-label="Build-up stages">
        <button v-for="(s,i) in stepLabels" :key="'dot'+i" class="dot"
                :class="{ 'is-active': current===i, 'is-done': current>i }"
                @click="goTo(i)" :aria-label="s" :aria-current="current===i"></button>
      </div>

      <div class="counter">STEP <b>{{ current+1 }}</b> / {{ stepLabels.length }}</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const dist = (x1,y1,x2,y2) => Math.hypot(x2-x1, y2-y1)

function arrowHead(x1,y1,x2,y2,size=10){
  const ang = Math.atan2(y2-y1, x2-x1)
  const a1 = ang + Math.PI*0.85
  const a2 = ang - Math.PI*0.85
  const p1x = (x2 + size*Math.cos(a1)).toFixed(1), p1y = (y2 + size*Math.sin(a1)).toFixed(1)
  const p2x = (x2 + size*Math.cos(a2)).toFixed(1), p2y = (y2 + size*Math.sin(a2)).toFixed(1)
  return `${x2},${y2} ${p1x},${p1y} ${p2x},${p2y}`
}

// ---- cone geometry (same scheme as JetDecayCones.vue) ----
const apex = { x:280, y:640 }
const rimY = 150
const svB = { x:280, y:545 }
const svC = { x:280, y:450 }
const flight1 = { x1:apex.x, y1:apex.y, x2:svB.x, y2:svB.y }
const flight2 = { x1:svB.x,  y1:svB.y,  x2:svC.x, y2:svC.y }

const promptTracks = [200,280,360].map(x => ({
  x1:apex.x, y1:apex.y, x2:x, y2:rimY, len:dist(apex.x,apex.y,x,rimY), color:'var(--accent)', group:1
}))
const svBTracks = [220,290,340].map(x => ({
  x1:svB.x, y1:svB.y, x2:x, y2:rimY, len:dist(svB.x,svB.y,x,rimY), color:'var(--rose)', group:2
}))
const svCTracks = [250,320].map(x => ({
  x1:svC.x, y1:svC.y, x2:x, y2:rimY, len:dist(svC.x,svC.y,x,rimY), color:'var(--orange)', group:3
}))
const allTracks = [...promptTracks, ...svBTracks, ...svCTracks]

// ---- token sequence laid out to the right ----
const BOX_W = 100, BOX_GAP = 14, START_X = 920, BOX_Y = 345
const tokens = allTracks.map((t,i) => ({
  label: 't'+(i+1),
  color: t.color,
  group: t.group,
  boxX: START_X + i*(BOX_W+BOX_GAP),
  boxY: BOX_Y,
}))
const seqCenterX = (tokens[0].boxX + tokens[tokens.length-1].boxX + BOX_W) / 2
const tokenRowBottom = BOX_Y + 100

// curved connector from each track's rim-exit point to its token box
const connectors = allTracks.map((t,i) => {
  const destX = tokens[i].boxX + 50, destY = tokens[i].boxY
  const midX = (t.x2 + destX) / 2
  const midY = (t.y2 + destY) / 2 - 35
  return {
    d: `M${t.x2},${t.y2} Q${midX},${midY} ${destX},${destY}`,
    len: dist(t.x2, t.y2, destX, destY),
    color: t.color,
    group: t.group,
  }
})

// ---- step 4: each token column becomes a small embedding vector ----
const MATRIX_ROWS = 6
const cellH = 24, cellGap = 4
const matrixStartY = tokenRowBottom + 50
tokens.forEach((tok, i) => {
  tok.cells = Array.from({ length: MATRIX_ROWS }, (_, j) => ({
    j,
    y: matrixStartY + j*(cellH+cellGap),
    opacity: 0.2 + (((i*5 + j*7) % 9) / 9) * 0.7,
    delay: i*0.05 + j*0.04,
  }))
})
const matrixEndY = matrixStartY + MATRIX_ROWS*(cellH+cellGap) - cellGap
const leftBracket = (() => {
  const x = tokens[0].boxX - 14, yTop = matrixStartY - 6, yBot = matrixEndY + 6
  return `M${x+10},${yTop} L${x},${yTop} L${x},${yBot} L${x+10},${yBot}`
})()
const rightBracket = (() => {
  const x = tokens[tokens.length-1].boxX + BOX_W + 14, yTop = matrixStartY - 6, yBot = matrixEndY + 6
  return `M${x-10},${yTop} L${x},${yTop} L${x},${yBot} L${x-10},${yBot}`
})()

// ---- step / reveal state ----
const stepLabels = ['Prompt tracks', 'SV(b) decay', 'SV(c) decay', 'Full sequence', 'Token matrix']
const current = ref(0)
const playing = ref(true)
let timer = null
const INTERVAL = 1900

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

onMounted(startTimer)
onBeforeUnmount(stopTimer)
</script>

<style scoped>
.jet-sequence{
  --bg: #0a0d12;
  --panel: #11161d;
  --panel-2: #161c25;
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
.jet-sequence *{ box-sizing: border-box; }

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

.panel-title{ font-size: 18px; font-weight: 600; fill: var(--text); }
.dim-label{
  font-family:'IBM Plex Mono', monospace;
  font-size: 13px;
  fill: var(--text-dim);
  letter-spacing: 0.03em;
  transition: opacity .4s ease;
}

.cone-outline{ fill:none; stroke: rgba(224,80,122,0.45); stroke-width: 3; stroke-linecap: round; }

.track{ fill:none; stroke-width: 3.4; stroke-linecap: round; transition: stroke-dashoffset .6s ease; }
.arrowhead{ transition: opacity .3s ease; }

.flight-line{ stroke: var(--lavender); stroke-width: 3; stroke-dasharray: 9 6; fill:none; transition: opacity .4s ease; }

.dot-pv{ fill: var(--red); stroke: #0a0d12; stroke-width: 2; }
.dot-svb{ fill: var(--rose); stroke: #0a0d12; stroke-width: 2; }
.dot-svc{ fill: var(--orange); stroke: #0a0d12; stroke-width: 2; }

.connector{ fill:none; stroke-width: 2; opacity: 0.7; transition: stroke-dashoffset .7s ease; }

.token{ transition: opacity .4s ease; }
.token-body{ fill: var(--panel-2); stroke: var(--border); }
.token-label{
  font-family:'IBM Plex Mono', monospace;
  font-size: 18px;
  font-weight: 600;
  fill: var(--text);
}

.matrix-cell{ stroke: #0a0d12; stroke-width: 1; transition: opacity .35s ease; }
.bracket{ fill:none; stroke: var(--text-dim); stroke-width: 3; transition: opacity .4s ease; }

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
</style>
