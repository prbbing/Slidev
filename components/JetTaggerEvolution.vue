<!--
  JetTaggerEvolution.vue
  Fourth companion piece in the set (alongside JetTaggerInference.vue,
  JetDecayCones.vue, JetEventDisplay.vue). Two rows of three stages,
  snaking left-to-right then left-to-right again: BDT -> Deep Neural Net
  -> RNN, then Deep Sets -> GNN -> Transformer. GNN and Transformer are
  both fully-connected graphs (that's accurate to how jet taggers actually
  use them) — the visual difference is how the edges behave: GNN's edges
  are uniform/structural, while the Transformer panel highlights one node
  attending to every other node with different, data-dependent weights
  (and the rest of the graph faded out), which is the actual distinction
  between message-passing and self-attention.

  Usage (e.g. in Slidev):
    ---
    layout: full
    ---
    <JetTaggerEvolution />

  Fills whatever box it's given (width:100%; height:100%).
-->
<template>
  <div class="evolution">
    <div class="stage">
      <svg viewBox="0 0 1400 770" xmlns="http://www.w3.org/2000/svg">

        <!-- straight intra-row and inter-row arrows -->
        <g v-for="(a,i) in straightArrows" :key="'arr'+i" class="arrow" :class="arrowClass(a.target)">
          <line :x1="a.x1" :y1="a.y1" :x2="a.x2" :y2="a.y2"/>
          <polygon :points="arrowHead(a.x1,a.y1,a.x2,a.y2)"/>
        </g>

        <!-- panel boxes + text -->
        <g v-for="(s,i) in stages" :key="'panel'+i" class="grp" :class="stateClass(i)">
          <rect class="fill-panel" :x="s.boxX" :y="s.boxY" width="400" height="230" rx="14"/>
          <text :x="s.cx" :y="s.boxY+256" text-anchor="middle" class="panel-title">{{ s.name }}</text>
          <text :x="s.cx" :y="s.boxY+276" text-anchor="middle" class="dim-label">{{ s.tag }}</text>
          <text :x="s.cx" :y="s.boxY+297" text-anchor="middle" class="year-label">{{ s.year }}</text>
        </g>

        <!-- ============ STAGE 0: BDT ============ -->
        <g class="grp" :class="stateClass(0)">
          <g class="glow-target" :transform="`translate(${pos.bdt.cx},${pos.bdt.cy})`">
            <line v-for="(l,i) in bdtLines" :key="'bl'+i" :x1="l[0].x" :y1="l[0].y" :x2="l[1].x" :y2="l[1].y" class="icon-bdt"/>
            <line :x1="bdtGhost1.root.x" :y1="bdtGhost1.root.y" :x2="bdtGhost1.kids[0].x" :y2="bdtGhost1.kids[0].y" class="icon-bdt ghost"/>
            <line :x1="bdtGhost1.root.x" :y1="bdtGhost1.root.y" :x2="bdtGhost1.kids[1].x" :y2="bdtGhost1.kids[1].y" class="icon-bdt ghost"/>
            <line :x1="bdtGhost2.root.x" :y1="bdtGhost2.root.y" :x2="bdtGhost2.kids[0].x" :y2="bdtGhost2.kids[0].y" class="icon-bdt ghost2"/>
            <line :x1="bdtGhost2.root.x" :y1="bdtGhost2.root.y" :x2="bdtGhost2.kids[1].x" :y2="bdtGhost2.kids[1].y" class="icon-bdt ghost2"/>
            <circle :cx="bdtRoot.x" :cy="bdtRoot.y" r="8" class="node-bdt"/>
            <circle v-for="(n,i) in bdtMid" :key="'bm'+i" :cx="n.x" :cy="n.y" r="8" class="node-bdt"/>
            <circle v-for="(n,i) in bdtLeaves" :key="'blf'+i" :cx="n.x" :cy="n.y" r="8" class="node-bdt"/>
          </g>
        </g>

        <!-- ============ STAGE 1: Deep Neural Net ============ -->
        <g class="grp" :class="stateClass(1)">
          <g class="glow-target" :transform="`translate(${pos.dnn.cx},${pos.dnn.cy})`">
            <line v-for="(e,i) in dnnEdges" :key="'de'+i" :x1="e[0].x" :y1="e[0].y" :x2="e[1].x" :y2="e[1].y" class="icon-nn"/>
            <circle v-for="(n,i) in dnnAllNodes" :key="'dn'+i" :cx="n.x" :cy="n.y" r="8" class="node-nn"/>
          </g>
        </g>

        <!-- ============ STAGE 2: RNN ============ -->
        <g class="grp" :class="stateClass(2)">
          <g class="glow-target" :transform="`translate(${pos.rnn.cx},${pos.rnn.cy})`">
            <path v-for="(n,i) in rnnNodes" :key="'rl'+i" :d="loopPath(n.x,n.y)" class="icon-nn loop"/>
            <line v-for="(a,i) in rnnArrows" :key="'ra'+i" :x1="a[0].x" :y1="a[0].y" :x2="a[1].x-14" :y2="a[1].y" class="icon-nn"/>
            <polygon v-for="(a,i) in rnnArrows" :key="'rah'+i" :points="arrowHead(a[0].x,a[0].y,a[1].x,a[1].y,10)" class="head-nn"/>
            <circle v-for="(n,i) in rnnNodes" :key="'rn'+i" :cx="n.x" :cy="n.y" r="9" class="node-nn"/>
          </g>
        </g>

        <!-- ============ STAGE 3: Deep Sets ============ -->
        <g class="grp" :class="stateClass(3)">
          <g class="glow-target" :transform="`translate(${pos.deepset.cx},${pos.deepset.cy})`">
            <line v-for="(n,i) in dsNodes" :key="'dsl'+i" :x1="n.x" :y1="n.y+10" :x2="dsAgg.x" :y2="dsAgg.y-14" class="icon-nn"/>
            <circle v-for="(n,i) in dsNodes" :key="'dsn'+i" :cx="n.x" :cy="n.y" r="8" class="node-nn"/>
            <circle :cx="dsAgg.x" :cy="dsAgg.y" r="15" class="node-agg"/>
          </g>
        </g>

        <!-- ============ STAGE 4: GNN (fully connected, uniform edges) ============ -->
        <g class="grp" :class="stateClass(4)">
          <g class="glow-target" :transform="`translate(${pos.gnn.cx},${pos.gnn.cy})`">
            <line v-for="(e,i) in hexEdges" :key="'ge'+i"
                  :x1="hexNodes[e[0]].x" :y1="hexNodes[e[0]].y" :x2="hexNodes[e[1]].x" :y2="hexNodes[e[1]].y"
                  class="icon-nn gnn-edge"/>
            <circle v-for="(n,i) in hexNodes" :key="'gn'+i" :cx="n.x" :cy="n.y" r="9" class="node-nn"/>
          </g>
        </g>

        <!-- ============ STAGE 5: Transformer (fully connected, weighted attention) ============ -->
        <g class="grp" :class="stateClass(5)">
          <g class="glow-target pulse-ring" :transform="`translate(${pos.transformer.cx},${pos.transformer.cy})`">
            <line v-for="(e,i) in hexEdges" :key="'te'+i"
                  :x1="hexNodes[e[0]].x" :y1="hexNodes[e[0]].y" :x2="hexNodes[e[1]].x" :y2="hexNodes[e[1]].y"
                  class="icon-nn"
                  :style="attnStyle(e[0], e[1])"/>
            <circle v-for="(n,i) in hexNodes" :key="'tn'+i" :cx="n.x" :cy="n.y" :r="i===0 ? 11 : 9" class="node-nn"/>
            <circle :cx="hexNodes[0].x" :cy="hexNodes[0].y" r="17" class="query-ring"/>
          </g>
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

      <div class="dots" role="tablist" aria-label="Architecture stages">
        <button v-for="(s,i) in stages" :key="'dot'+i" class="dot"
                :class="{ 'is-active': current===i, 'is-done': current>i }"
                @click="goTo(i)" :aria-label="s.name" :aria-current="current===i"></button>
      </div>

      <div class="counter">STEP <b>{{ current+1 }}</b> / {{ stages.length }}</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

function arrowHead(x1,y1,x2,y2,size=10){
  const ang = Math.atan2(y2-y1, x2-x1)
  const a1 = ang + Math.PI*0.85
  const a2 = ang - Math.PI*0.85
  const p1x = (x2 + size*Math.cos(a1)).toFixed(1), p1y = (y2 + size*Math.sin(a1)).toFixed(1)
  const p2x = (x2 + size*Math.cos(a2)).toFixed(1), p2y = (y2 + size*Math.sin(a2)).toFixed(1)
  return `${x2},${y2} ${p1x},${p1y} ${p2x},${p2y}`
}
function loopPath(x,y){
  return `M${x-11},${y-9} C${x-11},${y-32} ${x+11},${y-32} ${x+11},${y-9}`
}

// ---- panel layout: 2 rows x 3 columns ----
const pos = {
  bdt:         { cx:240,  cy:175, boxX:40,  boxY:60  },
  dnn:         { cx:700,  cy:175, boxX:500, boxY:60  },
  rnn:         { cx:1160, cy:175, boxX:960, boxY:60  },
  // row 2 runs right-to-left, so the column under RNN continues the chain
  // with a short drop instead of a long arrow back across the whole row
  deepset:     { cx:1160, cy:575, boxX:960, boxY:460 },
  gnn:         { cx:700,  cy:575, boxX:500, boxY:460 },
  transformer: { cx:240,  cy:575, boxX:40,  boxY:460 },
}

const stages = [
  { name:'BDT',              tag:'fixed splits',          year:'2012', ...pos.bdt },
  { name:'Deep Neural Net',  tag:'dense layers',          year:'2017', ...pos.dnn },
  { name:'RNN',              tag:'sequential',            year:'2018', ...pos.rnn },
  { name:'Deep Sets',        tag:'order-free, pooled',    year:'2019', ...pos.deepset },
  { name:'GNN',              tag:'fully-connected graph', year:'2020', ...pos.gnn },
  { name:'Transformer',      tag:'weighted attention',    year:'2022', ...pos.transformer },
]

// ---- arrows ----
// row 1 flows left-to-right, row 2 flows right-to-left (boustrophedon),
// so every connector — including between rows — is a short straight hop
// the row-1/row-2 connector sits in the same x-range as the centered
// title/tag/year text below RNN, so shift it sideways to clear the text
const wrapX = pos.rnn.cx + 110
const straightArrows = [
  { x1: pos.bdt.boxX+400, y1: pos.bdt.cy, x2: pos.dnn.boxX, y2: pos.dnn.cy, target: 1 },
  { x1: pos.dnn.boxX+400, y1: pos.dnn.cy, x2: pos.rnn.boxX, y2: pos.rnn.cy, target: 2 },
  { x1: wrapX, y1: pos.bdt.boxY+230, x2: wrapX, y2: pos.deepset.boxY, target: 3 },
  { x1: pos.deepset.boxX, y1: pos.deepset.cy, x2: pos.gnn.boxX+400, y2: pos.gnn.cy, target: 4 },
  { x1: pos.gnn.boxX, y1: pos.gnn.cy, x2: pos.transformer.boxX+400, y2: pos.transformer.cy, target: 5 },
]

// ---- Stage 0: BDT (a tree, plus two faint ensemble "ghost" trees) ----
const bdtRoot = { x:0, y:-70 }
const bdtMid = [ {x:-60,y:-20}, {x:60,y:-20} ]
const bdtLeaves = [ {x:-90,y:40}, {x:-30,y:40}, {x:30,y:40}, {x:90,y:40} ]
const bdtLines = [
  [bdtRoot, bdtMid[0]], [bdtRoot, bdtMid[1]],
  [bdtMid[0], bdtLeaves[0]], [bdtMid[0], bdtLeaves[1]],
  [bdtMid[1], bdtLeaves[2]], [bdtMid[1], bdtLeaves[3]],
]
const bdtGhost1 = { root:{x:-28,y:-50}, kids:[{x:-70,y:2},{x:8,y:2}] }
const bdtGhost2 = { root:{x:22,y:-42}, kids:[{x:-14,y:10},{x:58,y:10}] }

// ---- Stage 1: Deep Neural Net (fully-connected layers) ----
const dnnLayers = {
  input:  [-80,-27,27,80].map(y => ({ x:-130, y })),
  hidden: [-80,-27,27,80].map(y => ({ x:0,   y })),
  output: [-35,35].map(y => ({ x:130, y })),
}
const dnnAllNodes = [...dnnLayers.input, ...dnnLayers.hidden, ...dnnLayers.output]
const dnnEdges = []
dnnLayers.input.forEach(a => dnnLayers.hidden.forEach(b => dnnEdges.push([a,b])))
dnnLayers.hidden.forEach(a => dnnLayers.output.forEach(b => dnnEdges.push([a,b])))

// ---- Stage 2: RNN (a left-to-right chain with per-node memory loops) ----
const rnnNodes = [-150,-50,50,150].map(x => ({ x, y:0 }))
const rnnArrows = [[0,1],[1,2],[2,3]].map(([i,j]) => [rnnNodes[i], rnnNodes[j]])

// ---- Stage 3: Deep Sets (independent elements pooled into one summary) ----
const dsNodes = [-135,-45,45,135].map(x => ({ x, y:-65 }))
const dsAgg = { x:0, y:60 }

// ---- Stages 4 & 5 share the same fully-connected hexagon of nodes ----
const hexNodes = [
  { x:0,   y:-105 },
  { x:91,  y:-52.5 },
  { x:91,  y:52.5 },
  { x:0,   y:105 },
  { x:-91, y:52.5 },
  { x:-91, y:-52.5 },
]
const hexEdges = []
for (let i = 0; i < hexNodes.length; i++)
  for (let j = i+1; j < hexNodes.length; j++)
    hexEdges.push([i,j])

// node 0's attention weights to each other node — deliberately uneven and
// not just "nearest neighbour wins", since that's the whole point of
// attention vs. a structural graph
const attnWeights = { 1:0.85, 2:0.3, 3:0.7, 4:0.25, 5:0.55 }
function attnStyle(i,j){
  const other = i === 0 ? j : (j === 0 ? i : null)
  if (other === null) return { stroke: 'var(--text-faint)', strokeWidth: 1, opacity: 0.1 }
  const w = attnWeights[other]
  return { stroke: 'var(--accent)', strokeWidth: 1 + w*3, opacity: 0.25 + w*0.65 }
}

// ---- step state (same pattern as JetTaggerInference.vue) ----
const current = ref(0)
const playing = ref(true)
let timer = null
const INTERVAL = 1900

function advance(){ current.value = (current.value + 1) % stages.length }
function startTimer(){ stopTimer(); timer = setInterval(advance, INTERVAL) }
function stopTimer(){ if (timer) { clearInterval(timer); timer = null } }
function togglePlay(){ playing.value = !playing.value; playing.value ? startTimer() : stopTimer() }
function goTo(i){ current.value = i; playing.value = false; stopTimer() }
function step(dir){
  current.value = (current.value + dir + stages.length) % stages.length
  playing.value = false
  stopTimer()
}
function reset(){ current.value = 0; playing.value = true; startTimer() }

function stateClass(i){
  if (current.value === i) return 's-active'
  if (current.value > i) return 's-done'
  return 's-pending'
}
function arrowClass(targetIdx){
  if (current.value === targetIdx) return 'a-active'
  if (current.value > targetIdx) return 'a-done'
  return 'a-pending'
}

onMounted(startTimer)
onBeforeUnmount(stopTimer)
</script>

<style scoped>
.evolution{
  --bg: #0a0d12;
  --panel: #11161d;
  --panel-2: #161c25;
  --border: #232b37;
  --text: #e7eaf0;
  --text-dim: #828d9c;
  --text-faint: #4b5563;
  --accent: #5eead4;
  --accent-soft: rgba(94,234,212,0.18);
  --orange: #f5a524;

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
.evolution *{ box-sizing: border-box; }

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

.panel-title{ font-size: 17px; font-weight: 600; fill: var(--text); }
.dim-label{
  font-family:'IBM Plex Mono', monospace;
  font-size: 12px;
  fill: var(--text-dim);
  letter-spacing: 0.02em;
}
.year-label{
  font-family:'IBM Plex Mono', monospace;
  font-size: 12px;
  font-weight: 600;
  fill: var(--accent);
  letter-spacing: 0.03em;
}

.fill-panel{ fill: var(--panel-2); stroke: var(--border); }

.grp{ transition: opacity .45s ease; }
.grp.s-pending{ opacity: 0.28; }
.grp.s-done{ opacity: 1; }
.grp.s-active{ opacity: 1; }
.grp.s-active .glow-target{
  filter: drop-shadow(0 0 8px var(--accent)) drop-shadow(0 0 18px var(--accent-soft));
}
.grp.s-pending .glow-target, .grp.s-done .glow-target{ filter: none; }

.pulse-ring{ animation: pulse 1.7s ease-in-out infinite; }
@keyframes pulse{ 0%{opacity:.7;} 50%{opacity:1;} 100%{opacity:.7;} }

.arrow line, .arrow path{ stroke: var(--text-faint); stroke-width: 2; fill:none; transition: stroke .4s ease, opacity .4s ease; }
.arrow polygon{ fill: var(--text-faint); transition: fill .4s ease, opacity .4s ease; }
.arrow.a-done line, .arrow.a-done path{ stroke: #5b6573; }
.arrow.a-done polygon{ fill: #5b6573; }
.arrow.a-pending line, .arrow.a-pending path, .arrow.a-pending polygon{ opacity: .35; }
.arrow.a-active line, .arrow.a-active path{ stroke: var(--accent); stroke-width: 2.6; }
.arrow.a-active polygon{ fill: var(--accent); }

.icon-bdt{ stroke: var(--orange); stroke-width: 2.8; fill:none; }
.icon-bdt.ghost{ opacity: 0.3; stroke-width: 2; }
.icon-bdt.ghost2{ opacity: 0.22; stroke-width: 2; }
.node-bdt{ fill: var(--orange); stroke:#0a0d12; stroke-width:1.2; }

.icon-nn{ stroke: var(--accent); stroke-width: 1.8; opacity: 0.6; fill:none; }
.icon-nn.loop{ stroke-width: 2; opacity: 0.8; }
.gnn-edge{ stroke-width: 2; opacity: 0.55; }
.node-nn{ fill: var(--accent); stroke:#0a0d12; stroke-width:1.2; }
.node-agg{ fill: var(--panel-2); stroke: var(--accent); stroke-width: 2.6; }
.head-nn{ fill: var(--accent); }
.query-ring{ fill:none; stroke: var(--accent); stroke-width: 2; opacity: 0.8; }

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
  .pulse-ring{ animation: none !important; }
}
</style>
