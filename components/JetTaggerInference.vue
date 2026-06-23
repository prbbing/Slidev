<!--
  JetTaggerInference.vue
  Drop into any Vite / vue-cli project (Vue 3). Usage:

    <script setup>
    import JetTaggerInference from './JetTaggerInference.vue'
    </script>
    <template>
      <JetTaggerInference />
    </template>

  Fonts: this component assumes 'IBM Plex Sans' / 'IBM Plex Mono' are available.
  Either add the Google Fonts <link> tags to your index.html <head>, or keep
  the @import at the top of the <style> block below.
-->
<template>
  <div class="jet-tagger">

    <div class="stage">
      <svg viewBox="0 0 1760 700" xmlns="http://www.w3.org/2000/svg">

        <!-- ===================== ARROWS ===================== -->

        <g class="arrow" :class="arrowClass(1)">
          <line x1="330" y1="360" x2="412" y2="360"/>
          <polygon points="412,354 412,366 422,360"/>
        </g>

        <g class="arrow" :class="arrowClass(2)">
          <line x1="550" y1="360" x2="612" y2="360"/>
          <polygon points="612,354 612,366 622,360"/>
        </g>

        <g class="arrow" :class="arrowClass(3)">
          <line x1="730" y1="360" x2="792" y2="360"/>
          <polygon points="792,354 792,366 802,360"/>
        </g>

        <g class="arrow" :class="arrowClass(4)">
          <line x1="930" y1="360" x2="982" y2="360"/>
          <polygon points="982,354 982,366 992,360"/>
        </g>

        <g class="arrow" :class="arrowClass(5)">
          <line x1="1198" y1="360" x2="1272" y2="360"/>
          <polygon points="1272,354 1272,366 1282,360"/>
        </g>

        <g class="arrow" :class="arrowClass(6)">
          <path d="M1410,360 C1440,360 1440,195 1462,195"/>
          <polygon points="1456,189 1456,201 1466,195"/>
        </g>
        <g class="arrow" :class="arrowClass(6)">
          <line x1="1410" y1="360" x2="1462" y2="360"/>
          <polygon points="1456,354 1456,366 1466,360"/>
        </g>
        <g class="arrow" :class="arrowClass(6)">
          <path d="M1410,360 C1440,360 1440,525 1462,525"/>
          <polygon points="1456,519 1456,531 1466,525"/>
        </g>

        <g class="arrow" :class="arrowClass(7)">
          <line x1="1630" y1="195" x2="1652" y2="195"/>
          <polygon points="1652,190 1652,200 1660,195"/>
        </g>
        <g class="arrow" :class="arrowClass(7)">
          <line x1="1630" y1="360" x2="1652" y2="360"/>
          <polygon points="1652,355 1652,365 1660,360"/>
        </g>
        <g class="arrow" :class="arrowClass(7)">
          <line x1="1630" y1="525" x2="1652" y2="525"/>
          <polygon points="1652,520 1652,530 1660,525"/>
        </g>

        <!-- ===================== GROUP 0: inputs ===================== -->
        <g class="grp" :class="stateClass(0)">
          <g class="glow-target">
            <rect class="fill-lavender" x="40" y="335" width="110" height="50" rx="8"/>
            <text x="95" y="365" text-anchor="middle" class="block-sub" font-size="12">jet inputs</text>
          </g>
          <text x="95" y="403" text-anchor="middle" class="dim-label">n_jf</text>

          <g v-for="(c,i) in colors" :key="'ti'+i" class="glow-target">
            <rect :class="'fill-'+c" x="190" :y="rowY(i)" width="130" height="56" rx="6"/>
          </g>
          <text x="255" y="540" text-anchor="middle" class="dim-label">n_tracks × n_tf</text>
          <text x="255" y="170" text-anchor="middle" class="block-title">Track inputs</text>
        </g>

        <!-- ===================== GROUP 1: combined ===================== -->
        <g class="grp" :class="stateClass(1)">
          <g v-for="(c,i) in colors" :key="'cb'+i" class="glow-target">
            <rect class="fill-lavender" x="420" :y="rowY(i)" width="40" height="56" rx="6"/>
            <rect :class="'fill-'+c" x="462" :y="rowY(i)" width="88" height="56" rx="6"/>
          </g>
          <text x="485" y="170" text-anchor="middle" class="block-title">Combined inputs</text>
          <text x="485" y="540" text-anchor="middle" class="dim-label">n_tracks × (n_jf + n_tf)</text>
        </g>

        <!-- ===================== GROUP 2: initialiser ===================== -->
        <g class="grp" :class="stateClass(2)">
          <g class="glow-target">
            <rect class="fill-panel" x="620" y="270" width="110" height="180" rx="10"/>
            <text x="675" y="345" text-anchor="middle" class="block-title" font-size="13">Track</text>
            <text x="675" y="365" text-anchor="middle" class="block-title" font-size="13">initialiser</text>
          </g>
        </g>

        <!-- ===================== GROUP 3: initial track rep ===================== -->
        <g class="grp" :class="stateClass(3)">
          <g v-for="(c,i) in colors" :key="'it'+i" class="glow-target">
            <rect class="fill-lavender" x="800" :y="rowY(i)" width="20" height="56" rx="5"/>
            <rect :class="'fill-'+c" x="822" :y="rowY(i)" width="108" height="56" rx="6"/>
          </g>
          <text x="865" y="170" text-anchor="middle" class="block-title">Initial track</text>
          <text x="865" y="186" text-anchor="middle" class="block-title">representation</text>
        </g>

        <!-- ===================== GROUP 4: transformer ===================== -->
        <g class="grp" :class="stateClass(4)">
          <g class="glow-target pulse-ring">
            <circle class="fill-accent-box" cx="1090" cy="360" r="100"/>
          </g>
          <line v-for="(e,i) in transformerEdges" :key="'te'+i"
                class="edge-line"
                :x1="e.x1" :y1="e.y1" :x2="e.x2" :y2="e.y2" :stroke="colorHex(e.c)"/>
          <circle v-for="(n,i) in transformerNodes" :key="'nd'+i"
                  class="node-dot" :cx="n.x" :cy="n.y" r="9" :fill="colorHex(n.c)"/>
          <text x="1090" y="490" text-anchor="middle" class="block-title">Transformer encoder</text>
          <text x="1090" y="506" text-anchor="middle" class="dim-label">self-attention over tracks</text>
        </g>

        <!-- ===================== GROUP 5: conditional track rep ===================== -->
        <g class="grp" :class="stateClass(5)">
          <g v-for="(c,i) in colors" :key="'ct'+i" class="glow-target">
            <rect :class="'fill-'+c" x="1280" :y="rowY(i)" width="130" height="56" rx="6"/>
          </g>
          <text x="1345" y="170" text-anchor="middle" class="block-title">Conditional track</text>
          <text x="1345" y="186" text-anchor="middle" class="block-title">representation</text>
        </g>

        <!-- ===================== GROUP 6: heads ===================== -->
        <g class="grp" :class="stateClass(6)">
          <g class="glow-target">
            <rect class="fill-panel" x="1462" y="150" width="168" height="90" rx="10"/>
            <text x="1546" y="190" text-anchor="middle" class="block-title" font-size="13">Jet flavour</text>
            <text x="1546" y="208" text-anchor="middle" class="block-title" font-size="13">network</text>
          </g>
          <g class="glow-target">
            <rect class="fill-panel" x="1462" y="315" width="168" height="90" rx="10"/>
            <text x="1546" y="355" text-anchor="middle" class="block-title" font-size="13">Track origin</text>
            <text x="1546" y="373" text-anchor="middle" class="block-title" font-size="13">network</text>
          </g>
          <g class="glow-target">
            <rect class="fill-panel" x="1462" y="480" width="168" height="90" rx="10"/>
            <text x="1546" y="520" text-anchor="middle" class="block-title" font-size="13">Vertex grouping</text>
            <text x="1546" y="538" text-anchor="middle" class="block-title" font-size="13">network</text>
          </g>
        </g>

        <!-- ===================== GROUP 7: outputs ===================== -->
        <g class="grp" :class="stateClass(7)">
          <g class="glow-target">
            <rect class="fill-gray" x="1660" y="175" width="22" height="40" rx="4"/>
            <rect class="fill-gray" x="1690" y="175" width="22" height="40" rx="4"/>
          </g>
          <text x="1696" y="232" text-anchor="middle" class="dim-label" font-size="10.5">b / c / light</text>

          <g v-for="(c,i) in colors" :key="'oo'+i" class="glow-target">
            <rect :class="'fill-'+c" x="1660" :y="300+i*22" width="55" height="16" rx="3"/>
          </g>
          <text x="1687" y="408" text-anchor="middle" class="dim-label" font-size="10.5">per track</text>

          <g v-for="(p,i) in vertexPairs" :key="'vg'+i" class="glow-target">
            <line v-if="p.linked" :x1="1674" :y1="485+i*26" :x2="1690" :y2="485+i*26" class="pair-link"/>
            <rect :class="p.a" x="1660" :y="478+i*26" width="14" height="14" rx="3"/>
            <rect :class="p.b" x="1690" :y="478+i*26" width="14" height="14" rx="3"/>
          </g>
          <text x="1687" y="560" text-anchor="middle" class="dim-label" font-size="10.5">pairwise check</text>
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

      <div class="dots" role="tablist" aria-label="Inference stages">
        <button v-for="(s,i) in steps" :key="'dot'+i" class="dot"
                :class="{ 'is-active': current===i, 'is-done': current>i }"
                @click="goTo(i)" :aria-label="s.label" :aria-current="current===i"></button>
      </div>

      <div class="counter">STEP <b>{{ current+1 }}</b> / {{ steps.length }}</div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const colors = ['purple', 'orange', 'green', 'blue', 'red']
const colorMap = {
  purple: '#9b7bf0', orange: '#f5a524', green: '#34d399', blue: '#4d8cf6', red: '#f0615f'
}
const colorHex = (c) => colorMap[c]

const rowY = (i) => 200 + i * 66

// Each tile illustrates one pairwise check from the vertex-grouping head.
// Matching fill colors + a connecting line = "predicted same vertex";
// mismatched colors with no line = "predicted different vertex".
const vertexPairs = [
  { a: 'fill-match', b: 'fill-match', linked: true },
  { a: 'fill-match2', b: 'fill-match2', linked: true },
  { a: 'fill-match', b: 'fill-match2', linked: false },
]

const transformerNodes = [
  { x: 1055, y: 320, c: 'purple' },
  { x: 1125, y: 318, c: 'orange' },
  { x: 1145, y: 380, c: 'blue' },
  { x: 1085, y: 408, c: 'green' },
  { x: 1040, y: 372, c: 'red' },
]
const transformerEdges = []
for (let i = 0; i < transformerNodes.length; i++) {
  for (let j = i + 1; j < transformerNodes.length; j++) {
    const n = transformerNodes[i], m = transformerNodes[j]
    transformerEdges.push({ x1: n.x, y1: n.y, x2: m.x, y2: m.y, c: (i + j) % 2 === 0 ? n.c : m.c })
  }
}

const steps = [
  { label: 'Jet & track inputs' },
  { label: 'Combined inputs' },
  { label: 'Track initialiser' },
  { label: 'Initial track representation' },
  { label: 'Transformer encoder' },
  { label: 'Conditional track representation' },
  { label: 'Task heads' },
  { label: 'Predictions' },
]

const current = ref(0)
const playing = ref(true)
let timer = null
const INTERVAL = 1900

function advance() {
  current.value = (current.value + 1) % steps.length
}
function startTimer() {
  stopTimer()
  timer = setInterval(advance, INTERVAL)
}
function stopTimer() {
  if (timer) { clearInterval(timer); timer = null }
}
function togglePlay() {
  playing.value = !playing.value
  if (playing.value) startTimer(); else stopTimer()
}
function goTo(i) {
  current.value = i
  playing.value = false
  stopTimer()
}
function step(dir) {
  current.value = (current.value + dir + steps.length) % steps.length
  playing.value = false
  stopTimer()
}
function reset() {
  current.value = 0
  playing.value = true
  startTimer()
}

function stateClass(i) {
  if (current.value === i) return 's-active'
  if (current.value > i) return 's-done'
  return 's-pending'
}
function arrowClass(targetIdx) {
  if (current.value === targetIdx) return 'a-active'
  if (current.value > targetIdx) return 'a-done'
  return 'a-pending'
}

onMounted(startTimer)
onBeforeUnmount(stopTimer)
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:wght@400;500;600;700&family=IBM+Plex+Mono:wght@400;500;600&display=swap');

.jet-tagger{
  --bg: #0a0d12;
  --panel: #11161d;
  --panel-2: #161c25;
  --border: #232b37;
  --text: #e7eaf0;
  --text-dim: #828d9c;
  --text-faint: #4b5563;
  --accent: #5eead4;
  --accent-soft: rgba(94,234,212,0.18);
  --lavender: #93a5c9;
  --purple: #9b7bf0;
  --orange: #f5a524;
  --green: #34d399;
  --blue: #4d8cf6;
  --red: #f0615f;
  --gray-ctx: #515b69;

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
.jet-tagger *{ box-sizing: border-box; }

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
.stage svg{
  display: block;
  width: 100%;
  height: 100%;
}

.block-title{ font-size: 14.5px; font-weight: 600; fill: var(--text); }
.block-sub{
  font-family:'IBM Plex Mono', monospace;
  font-size: 11px;
  fill: var(--text-dim);
  letter-spacing: 0.02em;
}
.dim-label{
  font-family:'IBM Plex Mono', monospace;
  font-size: 11.5px;
  fill: var(--text-faint);
}

.grp{ transition: opacity .45s ease; }
.grp.s-pending{ opacity: 0.22; }
.grp.s-done{ opacity: 1; }
.grp.s-active{ opacity: 1; }

.grp.s-active .glow-target{
  filter: drop-shadow(0 0 10px var(--accent)) drop-shadow(0 0 22px var(--accent-soft));
}
.grp.s-pending .glow-target, .grp.s-done .glow-target{ filter: none; }

.pulse-ring{
  animation: pulse 1.7s ease-in-out infinite;
  transform-origin: center;
}
@keyframes pulse{
  0%{ opacity:.55; }
  50%{ opacity: 1; }
  100%{ opacity:.55; }
}

.arrow line, .arrow path{
  stroke: var(--text-faint);
  stroke-width: 2;
  fill: none;
  transition: stroke .4s ease, opacity .4s ease;
}
.arrow polygon{
  fill: var(--text-faint);
  transition: fill .4s ease, opacity .4s ease;
}
.arrow.a-done line, .arrow.a-done path{ stroke: #5b6573; }
.arrow.a-done polygon{ fill: #5b6573; }
.arrow.a-pending line, .arrow.a-pending path{ opacity: .35; }
.arrow.a-pending polygon{ opacity: .35; }
.arrow.a-active line, .arrow.a-active path{
  stroke: var(--accent);
  stroke-width: 2.6;
  stroke-dasharray: 7 6;
  animation: flow 0.6s linear infinite;
  filter: drop-shadow(0 0 6px var(--accent-soft));
}
.arrow.a-active polygon{ fill: var(--accent); }
@keyframes flow{
  to{ stroke-dashoffset: -26; }
}

rect, circle{ stroke-width: 1.6; }
.fill-lavender{ fill: rgba(147,165,201,0.22); stroke: var(--lavender); }
.fill-purple{ fill: rgba(155,123,240,0.22); stroke: var(--purple); }
.fill-orange{ fill: rgba(245,165,36,0.20); stroke: var(--orange); }
.fill-green{ fill: rgba(52,211,153,0.20); stroke: var(--green); }
.fill-blue{ fill: rgba(77,140,246,0.22); stroke: var(--blue); }
.fill-red{ fill: rgba(240,97,95,0.20); stroke: var(--red); }
.fill-gray{ fill: rgba(81,91,105,0.35); stroke: var(--gray-ctx); }
.fill-panel{ fill: var(--panel-2); stroke: var(--border); }
.fill-accent-box{ fill: rgba(94,234,212,0.08); stroke: var(--accent); }
.fill-match{ fill: rgba(94,234,212,0.38); stroke: var(--accent); }
.fill-match2{ fill: rgba(155,123,240,0.32); stroke: var(--purple); }
.pair-link{ stroke: var(--text-dim); stroke-width: 2; }

.node-dot{ stroke: #0a0d12; stroke-width: 1.5; }
.edge-line{ stroke-width: 1.4; opacity: 0.85; }

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
  background: var(--panel-2);
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
  .pulse-ring, .arrow.a-active line, .arrow.a-active path{ animation: none !important; }
}
</style>
