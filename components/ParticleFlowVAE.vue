<!--
  ParticleFlowVAE.vue
  Combines the PFN/DeepSets classifier and the VAE into one continuous
  pipeline that shares a single pooled representation O. The classifier
  now sits in the same row as the particle pipeline (continuing right
  after O, at the same height as Φ), since classifier and VAE are both
  one-shot heads reading O — only the VAE actually needs to go elsewhere
  on the canvas (downward), since it's a multi-stage chain rather than a
  single box. Canvas is 1460x1150 — wider than tall, so this one suits a
  full-width slide better than a half-width column.

  Usage (e.g. in Slidev, as the main content of a slide):
    ---
    layout: full
    ---
    <ParticleFlowVAE />

  Fills whatever box it's given (width:100%; height:100%), so it still
  works in a half-column if you want it there too — it'll just render
  smaller, with empty space left/right rather than top/bottom.
-->
<template>
  <div class="pfn-vae">
    <div class="stage">
      <svg viewBox="0 0 1460 1150" xmlns="http://www.w3.org/2000/svg">

        <text x="725" y="50" text-anchor="middle" class="section-title">PFN — DeepSets architecture, branching into a classifier and a VAE</text>

        <!-- ===== particle inputs ===== -->
        <g class="grp" :class="stateClass(0)">
          <text x="135" y="85" text-anchor="middle" class="block-title">Particle inputs</text>
          <g v-for="(c,i) in colors" :key="'pi'+i" class="glow-target">
            <rect :class="'fill-'+c" x="40" :y="rowY(i)" width="190" height="58" rx="8"/>
          </g>
          <text x="135" y="475" text-anchor="middle" class="dim-label">× many particles</text>
        </g>

        <!-- ===== shared Φ network + per-particle embeddings ===== -->
        <g class="arrow" :class="arrowClass(1)">
          <line x1="230" y1="275" x2="252" y2="275"/>
          <polygon :points="arrowHead(230,275,260,275)"/>
        </g>
        <g class="grp" :class="stateClass(1)">
          <g class="glow-target">
            <rect class="fill-panel" x="260" y="110" width="150" height="330" rx="14"/>
            <text x="335" y="265" text-anchor="middle" class="block-title" font-size="20">Network</text>
            <text x="335" y="295" text-anchor="middle" class="block-title" font-size="20">Φ</text>
          </g>
          <text x="335" y="475" text-anchor="middle" class="dim-label">shared weights</text>
        </g>
        <g class="arrow" :class="arrowClass(1)">
          <line x1="410" y1="275" x2="432" y2="275"/>
          <polygon :points="arrowHead(410,275,440,275)"/>
        </g>
        <g class="grp" :class="stateClass(1)">
          <text x="535" y="85" text-anchor="middle" class="block-title">Per-particle embeddings</text>
          <g v-for="(c,i) in colors" :key="'pe'+i" class="glow-target">
            <rect :class="'fill-'+c" x="440" :y="rowY(i)" width="190" height="58" rx="8"/>
          </g>
        </g>

        <!-- ===== sum pooling -> O ===== -->
        <g class="grp" :class="stateClass(2)">
          <line v-for="(c,i) in colors" :key="'sum'+i" class="sum-line glow-target"
                x1="630" :y1="rowY(i)+29" x2="654" y2="275"/>
          <circle cx="680" cy="275" r="30" class="agg-node glow-target"/>
          <text x="680" y="284" text-anchor="middle" class="agg-symbol">Σ</text>
          <g class="arrow">
            <line x1="710" y1="275" x2="722" y2="275"/>
            <polygon :points="arrowHead(710,275,732,275)"/>
          </g>
          <text x="818.5" y="155" text-anchor="middle" class="col-label">Pooled representation O</text>
          <rect v-for="cell in oCells" :key="'o'+cell.j"
                x="736" :y="cell.y" width="165" height="28" rx="5" class="o-cell glow-target"
                :style="{ opacity: cell.opacity }"/>
        </g>

        <!-- ===== branch A: classifier F, continuing the same row as O (top-aligned with Φ at y=110) ===== -->
        <g class="grp" :class="stateClass(3)">
          <g class="arrow">
            <line x1="901" y1="275" x2="1030" y2="275"/>
            <polygon :points="arrowHead(901,275,1050,275)"/>
          </g>
          <g class="glow-target">
            <rect class="fill-panel" x="1050" y="110" width="260" height="330" rx="14"/>
            <text x="1180" y="283" text-anchor="middle" class="block-title" font-size="20">Classifier F</text>
          </g>
          <g class="arrow">
            <line x1="1310" y1="270" x2="1340" y2="270"/>
            <polygon :points="arrowHead(1310,270,1360,270)"/>
          </g>
          <rect x="1360" y="245" width="50" height="50" rx="8" class="out-swatch glow-target"/>
          <rect x="1360" y="305" width="50" height="50" rx="8" class="out-swatch glow-target"/>
          <text x="1385" y="385" text-anchor="middle" class="dim-label">signal /</text>
          <text x="1385" y="403" text-anchor="middle" class="dim-label">background</text>
        </g>

        <!-- ===== branch B: VAE, fed straight down from the same O ===== -->
        <g class="grp" :class="stateClass(4)">
          <g class="arrow">
            <line x1="818.5" y1="369" x2="818.5" y2="400"/>
            <polygon :points="arrowHead(818.5,369,818.5,410)"/>
          </g>
          <g class="glow-target">
            <rect class="fill-panel" x="688.5" y="410" width="260" height="150" rx="14"/>
            <text x="818.5" y="495" text-anchor="middle" class="block-title" font-size="20">Encoder</text>
          </g>
          <g class="arrow">
            <line x1="818.5" y1="560" x2="818.5" y2="572"/>
            <polygon :points="arrowHead(818.5,560,818.5,582)"/>
          </g>
          <text x="906" y="644" class="col-label">Latent z</text>
          <rect v-for="cell in zCells" :key="'z'+cell.j"
                x="746" :y="cell.y" width="145" height="34" rx="5" class="z-cell glow-target"
                :style="{ opacity: cell.opacity }"/>
        </g>

        <!-- ===== VAE decoder ===== -->
        <g class="grp" :class="stateClass(5)">
          <g class="arrow">
            <line x1="818.5" y1="696" x2="818.5" y2="708"/>
            <polygon :points="arrowHead(818.5,696,818.5,718)"/>
          </g>
          <g class="glow-target">
            <rect class="fill-panel" x="688.5" y="718" width="260" height="150" rx="14"/>
            <text x="818.5" y="803" text-anchor="middle" class="block-title" font-size="20">Decoder</text>
          </g>
          <g class="arrow">
            <line x1="818.5" y1="868" x2="818.5" y2="880"/>
            <polygon :points="arrowHead(818.5,868,818.5,890)"/>
          </g>
          <text x="916" y="989" class="col-label">Reconstruction Ô</text>
          <rect v-for="cell in reconCells" :key="'r'+cell.j"
                x="736" :y="cell.y" width="165" height="28" rx="5" class="recon-cell glow-target"
                :style="{ opacity: cell.opacity }"/>
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

function arrowHead(x1,y1,x2,y2,size=12){
  const ang = Math.atan2(y2-y1, x2-x1)
  const a1 = ang + Math.PI*0.85
  const a2 = ang - Math.PI*0.85
  const p1x = (x2 + size*Math.cos(a1)).toFixed(1), p1y = (y2 + size*Math.sin(a1)).toFixed(1)
  const p2x = (x2 + size*Math.cos(a2)).toFixed(1), p2y = (y2 + size*Math.sin(a2)).toFixed(1)
  return `${x2},${y2} ${p1x},${p1y} ${p2x},${p2y}`
}
const rowY = (i) => 110 + i*68
const colors = ['purple', 'orange', 'green', 'blue', 'red']

function makeCells(n, h, gap, centerY){
  const total = n*h + (n-1)*gap
  const startY = centerY - total/2
  return Array.from({ length:n }, (_,j) => ({
    j, y: startY + j*(h+gap),
    opacity: 0.25 + (((j*5+3) % 9)/9)*0.65,
  }))
}
const oCells = makeCells(6, 28, 4, 275)
const zCells = makeCells(3, 34, 6, 639)
const reconCells = makeCells(6, 28, 4, 984)

const stepLabels = [
  'Particle inputs',
  'Shared network Φ',
  'Sum pooling → O',
  'Classifier F',
  'VAE encoder',
  'VAE decoder',
]
const current = ref(0)
const playing = ref(true)
let timer = null
const INTERVAL = 1900

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

function stateClass(i){
  if (current.value === i) return 's-active'
  if (current.value > i) return 's-done'
  return 's-pending'
}
function arrowClass(i){
  if (current.value === i) return 'a-active'
  if (current.value > i) return 'a-done'
  return 'a-pending'
}

onMounted(startTimer)
onBeforeUnmount(stopTimer)
</script>

<style scoped>
.pfn-vae{
  --bg: #0a0d12;
  --panel: #11161d;
  --panel-2: #161c25;
  --border: #232b37;
  --text: #e7eaf0;
  --text-dim: #828d9c;
  --text-faint: #4b5563;
  --accent: #5eead4;
  --accent-soft: rgba(94,234,212,0.18);
  --green-latent: #22c55e;

  background: var(--bg);
  color: var(--text);
  font-family: 'IBM Plex Sans', sans-serif;
  -webkit-font-smoothing: antialiased;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 14px 20px;
}
.pfn-vae *{ box-sizing: border-box; }

.stage{
  flex: 1 1 auto;
  min-height: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--panel);
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 8px;
  overflow: hidden;
}
.stage svg{ display:block; width:100%; height:100%; }

.section-title{ font-size: 24px; font-weight: 700; fill: var(--text-dim); letter-spacing: 0.02em; }
.block-title{ font-size: 18px; font-weight: 600; fill: var(--text); }
.col-label{ font-size: 17px; font-weight: 600; fill: var(--text); }
.dim-label{
  font-family:'IBM Plex Mono', monospace;
  font-size: 14px;
  fill: var(--text-dim);
  letter-spacing: 0.02em;
}

.grp{ transition: opacity .45s ease; }
.grp.s-pending{ opacity: 0.22; }
.grp.s-done, .grp.s-active{ opacity: 1; }
.grp.s-active .glow-target{
  filter: drop-shadow(0 0 9px var(--accent)) drop-shadow(0 0 20px var(--accent-soft));
}
.grp.s-pending .glow-target, .grp.s-done .glow-target{ filter: none; }

.arrow line, .arrow path{ stroke: var(--text-faint); stroke-width: 3; fill:none; transition: stroke .4s ease, opacity .4s ease; }
.arrow polygon{ fill: var(--text-faint); transition: fill .4s ease, opacity .4s ease; }
.grp.s-active .arrow line, .grp.s-done .arrow line,
.grp.s-active .arrow path, .grp.s-done .arrow path{ stroke: var(--accent); }
.grp.s-active .arrow polygon, .grp.s-done .arrow polygon{ fill: var(--accent); }
.grp.s-pending .arrow line, .grp.s-pending .arrow path, .grp.s-pending .arrow polygon{ opacity: .3; }

.arrow.a-active line{ stroke: var(--accent); stroke-width: 3.4; }
.arrow.a-active polygon{ fill: var(--accent); }
.arrow.a-done line{ stroke: #5b6573; }
.arrow.a-done polygon{ fill: #5b6573; }
.arrow.a-pending line, .arrow.a-pending polygon{ opacity: .3; }

.fill-panel{ fill: var(--panel-2); stroke: var(--border); }
.fill-purple{ fill: rgba(155,123,240,0.28); stroke: #9b7bf0; }
.fill-orange{ fill: rgba(245,165,36,0.26); stroke: #f5a524; }
.fill-green{  fill: rgba(52,211,153,0.26); stroke: #34d399; }
.fill-blue{   fill: rgba(77,140,246,0.28); stroke: #4d8cf6; }
.fill-red{    fill: rgba(240,97,95,0.26);  stroke: #f0615f; }

.sum-line{ stroke: var(--text-faint); stroke-width: 2; opacity: 0.7; fill:none; }
.agg-node{ fill: var(--panel-2); stroke: var(--accent); stroke-width: 3; }
.agg-symbol{ font-size: 28px; font-weight: 700; fill: var(--accent); text-anchor: middle; }

.o-cell, .recon-cell{ fill: var(--accent); stroke: #0a0d12; stroke-width: 1; }
.recon-cell{ stroke: var(--accent); stroke-dasharray: 5 3; fill: rgba(94,234,212,0.15); }
.z-cell{ fill: var(--green-latent); stroke: #0a0d12; stroke-width: 1; }

.out-swatch{ fill: #cfd6e0; stroke: #cfd6e0; }

.controls{
  flex: 0 0 auto;
  display: flex;
  align-items: center;
  gap: 12px;
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
  padding: 7px 12px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  transition: border-color .2s, color .2s;
}
.btn:hover{ border-color: var(--accent); color: var(--accent); }
.btn:focus-visible{ outline: 2px solid var(--accent); outline-offset: 2px; }
.btn.play{ background: var(--accent); color: #06231e; border-color: var(--accent); font-weight:600; }
.btn.play:hover{ color:#06231e; }

.dots{ display:flex; gap:6px; align-items:center; }
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
  font-size: 11px;
  color: var(--text-dim);
  margin-left: auto;
}
.counter b{ color: var(--accent); }
</style>
