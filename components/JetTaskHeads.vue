<!--
  JetTaskHeads.vue
  Standalone extract from JetTaggerInference.vue: just the three parallel
  task heads (jet flavour / track origin / vertex grouping) and their
  outputs, stacked vertically. No play/pause/step controls — this is a
  static, always-on diagram (with a gentle ambient glow/flow animation),
  since the three heads run in parallel rather than as a sequence.

  Layout is compact and every output block — regardless of its own shape
  or width — is centered on the same shared horizontal axis, so the three
  rows read as one consistent column rather than three loosely related
  pieces.

  Usage (e.g. in Slidev):
    ---
    layout: full
    ---
    <JetTaskHeads />

  Fills whatever box it's given (width:100%; height:100%).
-->
<template>
  <div class="task-heads">
    <div class="stage">
      <svg :viewBox="`0 0 ${VB_W} ${VB_H}`" xmlns="http://www.w3.org/2000/svg">

        <g v-for="(row,i) in rows" :key="'row'+i">
          <rect class="fill-panel" :x="BOX_X" :y="row.boxY" :width="BOX_W" :height="BOX_H" rx="12"/>
          <text :x="BOX_X+BOX_W/2" :y="row.boxY+57" text-anchor="middle" class="head-title">{{ row.line1 }}</text>
          <text v-if="row.line2" :x="BOX_X+BOX_W/2" :y="row.boxY+83" text-anchor="middle" class="head-title">{{ row.line2 }}</text>

          <g class="arrow">
            <line :x1="ARROW_X1" :y1="row.centerY" :x2="ARROW_X2-10" :y2="row.centerY"/>
            <polygon :points="`${ARROW_X2-10},${row.centerY-6} ${ARROW_X2-10},${row.centerY+6} ${ARROW_X2},${row.centerY}`"/>
          </g>
        </g>

        <!-- row 0 output: jet flavour classes -->
        <rect v-for="(s,i) in flavourSquares" :key="'fs'+i"
              :x="s.x" :y="rows[0].centerY-18" width="36" height="36" rx="8" class="swatch flavour-swatch"/>
        <text :x="zoneCenterX" :y="rows[0].centerY+36" text-anchor="middle" class="dim-label">b / c / light</text>

        <!-- row 1 output: per-track colour bars -->
        <rect v-for="(b,i) in trackBars" :key="'tb'+i"
              :x="b.x" :y="b.y" width="124" height="20" rx="6" class="swatch" :style="{ fill: b.color, stroke: b.color }"/>
        <text :x="zoneCenterX" :y="rows[1].centerY+78" text-anchor="middle" class="dim-label">per track</text>

        <!-- row 2 output: pairwise vertex check -->
        <g v-for="(p,i) in vertexPairs" :key="'vp'+i">
          <line v-if="p.linked" :x1="p.x+20" :y1="p.y+10" :x2="p.x+28" :y2="p.y+10" class="pair-link"/>
          <rect :x="p.x" :y="p.y" width="20" height="20" rx="4" class="swatch" :style="{ fill: p.a, stroke: p.a }"/>
          <rect :x="p.x+28" :y="p.y" width="20" height="20" rx="4" class="swatch" :style="{ fill: p.b, stroke: p.b }"/>
        </g>
        <text :x="zoneCenterX" :y="rows[2].centerY+58" text-anchor="middle" class="dim-label">pairwise check</text>

      </svg>
    </div>
  </div>
</template>

<script setup>
// ---- compact layout constants ----
const ROW_H = 170, GAP = 25, TOP_MARGIN = 20
const BOX_X = 20, BOX_W = 240, BOX_H = 140
const ARROW_X1 = BOX_X + BOX_W, ARROW_X2 = ARROW_X1 + 40
const ZONE_X = 310, ZONE_W = 180
const VB_W = 520

const rowMeta = [
  { line1: 'Jet flavour',     line2: 'network' },
  { line1: 'Track origin',    line2: 'network' },
  { line1: 'Vertex grouping', line2: 'network' },
]
const rows = rowMeta.map((m, i) => {
  const y = TOP_MARGIN + i*(ROW_H+GAP)
  return { ...m, y, boxY: y + (ROW_H-BOX_H)/2, centerY: y + ROW_H/2 }
})
const VB_H = rows[rows.length-1].y + ROW_H + TOP_MARGIN

const zoneCenterX = ZONE_X + ZONE_W/2

// row 0: three output classes (b / c / light), centered in the zone
const flavourTotalW = 3*36 + 2*8
const flavourStartX = ZONE_X + (ZONE_W-flavourTotalW)/2
const flavourSquares = [0,1,2].map(i => ({ x: flavourStartX + i*44 }))

// row 1: five per-track colour bars, sharing the same centered column
const trackColors = ['#9b7bf0', '#f5a524', '#34d399', '#4d8cf6', '#f0615f']
const trackBars = trackColors.map((color,i) => ({ color, x: flavourStartX, y: rows[1].centerY - 60 + i*25 }))

// row 2: three pairwise-compatibility tiles, centered in the zone
const tileW = 20+8+20
const tileStartX = ZONE_X + (ZONE_W-tileW)/2
const accent = '#5eead4', purple = '#9b7bf0'
const vertexPairs = [
  { a: accent, b: accent, linked: true,  x: tileStartX, y: rows[2].centerY - 40 },
  { a: purple, b: purple, linked: true,  x: tileStartX, y: rows[2].centerY - 10 },
  { a: accent, b: purple, linked: false, x: tileStartX, y: rows[2].centerY + 20 },
]
</script>

<style scoped>
.task-heads{
  --bg: #0a0d12;
  --panel-2: #161c25;
  --border: #232b37;
  --text: #e7eaf0;
  --text-dim: #828d9c;
  --accent: #5eead4;
  --accent-soft: rgba(94,234,212,0.25);

  background: var(--bg);
  color: var(--text);
  font-family: 'IBM Plex Sans', sans-serif;
  -webkit-font-smoothing: antialiased;
  width: 100%;
  height: 100%;
  display: flex;
  padding: 18px 28px;
}
.task-heads *{ box-sizing: border-box; }

.stage{
  flex: 1 1 auto;
  min-height: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #11161d;
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 10px;
  overflow: hidden;
}
.stage svg{ display:block; width:100%; height:100%; }

.fill-panel{ fill: var(--panel-2); stroke: var(--border); }
.head-title{ font-size: 17px; font-weight: 600; fill: var(--text); }
.dim-label{
  font-family:'IBM Plex Mono', monospace;
  font-size: 12px;
  fill: var(--text-dim);
  letter-spacing: 0.02em;
}

.arrow line{
  stroke: var(--accent);
  stroke-width: 3;
  stroke-dasharray: 6 5;
  animation: flow 0.7s linear infinite;
}
.arrow polygon{ fill: var(--accent); }
@keyframes flow{ to{ stroke-dashoffset: -22; } }

.swatch{
  stroke-width: 1.4;
  filter: drop-shadow(0 0 6px var(--accent-soft));
  animation: breathe 2.6s ease-in-out infinite;
}
.flavour-swatch{ fill: #cfd6e0; stroke: #cfd6e0; }
.pair-link{ stroke: var(--text-dim); stroke-width: 2; }

@keyframes breathe{ 0%,100%{ opacity: 0.82; } 50%{ opacity: 1; } }

@media (prefers-reduced-motion: reduce){
  .arrow line, .swatch{ animation: none !important; }
}
</style>
