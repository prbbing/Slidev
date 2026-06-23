<!--
  LHCRing.vue
  Standalone illustrative animation of the LHC: a 27 km ring with two
  beams continuously orbiting in opposite directions, colliding at one
  marked interaction point (a brief particle-spray flash, reusing the
  same palette as the JetEventDisplay component for visual continuity).

  Purely ambient/looping — no play/pause controls needed, since there's
  no sequence to step through, just continuous motion during the talk.

  Usage (e.g. in Slidev):
    ---
    layout: full
    ---
    <LHCRing />

  Fills whatever box it's given (width:100%; height:100%).
-->
<template>
  <div class="lhc-ring">
    <div class="stage">
      <svg viewBox="0 0 900 900" xmlns="http://www.w3.org/2000/svg">

        <text x="450" y="50" text-anchor="middle" class="title">Large Hadron Collider</text>

        <!-- the 27 km ring -->
        <circle cx="450" cy="450" r="320" class="ring"/>
        <text x="450" y="878" text-anchor="middle" class="dim-label">27 km circumference</text>

        <!-- collision point marker -->
        <circle cx="450" cy="130" r="20" class="ip-marker"/>
        <text x="540" y="126" class="dim-label">Collision</text>
        <text x="540" y="150" class="dim-label">point</text>

        <!-- particle burst, fires once per orbit -->
        <g class="collision-burst">
          <circle cx="450" cy="130" r="14" class="burst-flash"/>
          <line v-for="(l,i) in burstLines" :key="'bl'+i"
                x1="450" y1="130" :x2="l.x2" :y2="l.y2"
                class="burst-line" :style="{ stroke: l.color }"/>
        </g>

        <!-- two counter-rotating beams, both starting at the collision point -->
        <circle cx="450" cy="130" r="11" class="beam beam-a"/>
        <circle cx="450" cy="130" r="11" class="beam beam-b"/>

        <!-- legend -->
        <circle cx="60" cy="792" r="8" class="beam-a-dot"/>
        <text x="84" y="798" class="dim-label">Beam 1 (clockwise)</text>
        <circle cx="60" cy="828" r="8" class="beam-b-dot"/>
        <text x="84" y="834" class="dim-label">Beam 2 (counter-clockwise)</text>

      </svg>
    </div>
  </div>
</template>

<script setup>
// 8 short "shrapnel" lines radiating from the collision point, cycling
// through the same track-colour palette used in JetEventDisplay.vue
const palette = ['#9b7bf0', '#f5a524', '#34d399', '#4d8cf6', '#f0615f', '#5eead4', '#e0507a', '#93a5c9']
const burstLines = Array.from({ length: 8 }, (_, i) => {
  const angle = (i / 8) * Math.PI * 2
  const len = 50 + (i % 3) * 10
  return {
    x2: 450 + Math.cos(angle) * len,
    y2: 130 + Math.sin(angle) * len,
    color: palette[i],
  }
})
</script>

<style scoped>
.lhc-ring{
  --bg: #0a0d12;
  --border: #232b37;
  --text: #e7eaf0;
  --text-dim: #828d9c;
  --accent: #5eead4;
  --lavender: #93a5c9;
  --rose: #e0507a;

  background: var(--bg);
  color: var(--text);
  font-family: 'IBM Plex Sans', sans-serif;
  -webkit-font-smoothing: antialiased;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 18px 28px;
}
.lhc-ring *{ box-sizing: border-box; }

.stage{
  width: 100%;
  max-height: 100%;
  aspect-ratio: 1 / 1;
  background: #11161d;
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 10px;
  overflow: hidden;
}
.stage svg{ display:block; width:100%; height:100%; }

.title{ font-size: 30px; font-weight: 600; fill: var(--text); }
.dim-label{
  font-family:'IBM Plex Mono', monospace;
  font-size: 17px;
  fill: var(--text-dim);
  letter-spacing: 0.02em;
}

.ring{
  fill: none;
  stroke: var(--lavender);
  stroke-width: 8;
  opacity: 0.55;
}

.ip-marker{
  fill: none;
  stroke: var(--text-dim);
  stroke-width: 2;
  stroke-dasharray: 4 4;
}

.beam{ stroke: #0a0d12; stroke-width: 1.5; filter: drop-shadow(0 0 8px currentColor); }
.beam-a{ fill: var(--accent); color: var(--accent); animation: orbitCW 5s linear infinite; transform-origin: 450px 450px; }
.beam-b{ fill: var(--rose);   color: var(--rose);   animation: orbitCCW 5s linear infinite; transform-origin: 450px 450px; }
.beam-a-dot{ fill: var(--accent); }
.beam-b-dot{ fill: var(--rose); }

@keyframes orbitCW  { from{ transform: rotate(0deg);   } to{ transform: rotate(360deg);  } }
@keyframes orbitCCW { from{ transform: rotate(0deg);   } to{ transform: rotate(-360deg); } }

.collision-burst{
  animation: burstFade 5s ease-out infinite;
  transform-origin: 450px 130px;
}
.burst-flash{ fill: #fff8e6; }
.burst-line{ stroke-width: 3; stroke-linecap: round; fill:none; }

@keyframes burstFade{
  0%   { opacity: 1; transform: scale(0.4); }
  18%  { opacity: 0; transform: scale(2.3); }
  100% { opacity: 0; transform: scale(2.3); }
}

@media (prefers-reduced-motion: reduce){
  .beam-a, .beam-b, .collision-burst{ animation: none !important; }
}
</style>
