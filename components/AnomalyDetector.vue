<script setup>
import { onMounted } from 'vue'

let renderFn = null

onMounted(() => {
  const NS = 'http://www.w3.org/2000/svg'
  function mulberry32(a){return function(){a|=0;a=a+0x6D2B79F5|0;let t=Math.imul(a^a>>>15,1|a);t=t+Math.imul(t^t>>>7,61|t)^t;return((t^t>>>14)>>>0)/4294967296;}}
  const rand = mulberry32(424242)
  function gaussPair(){let u1=Math.max(rand(),1e-6),u2=rand();const r=Math.sqrt(-2*Math.log(u1));const th=2*Math.PI*u2;return[r*Math.cos(th),r*Math.sin(th)];}
  const BG_C=[190,150], BG_S=52, SIG_C=[520,75], SIG_S=14, DECAY=90
  function sampleBg(){let p;do{p=gaussPair();}while(Math.hypot(p[0],p[1])>2.2);return[BG_C[0]+p[0]*BG_S,BG_C[1]+p[1]*BG_S];}
  function sampleSig(){let p;do{p=gaussPair();}while(Math.hypot(p[0],p[1])>2.2);return[SIG_C[0]+p[0]*SIG_S,SIG_C[1]+p[1]*SIG_S];}
  function sampleBgMass(){let m;do{m=100-Math.log(Math.max(rand(),1e-6))*DECAY;}while(m>500);return m;}
  function sampleSigMass(){let m;do{const p=gaussPair();m=250+p[0]*6;}while(m<100||m>500);return m;}
  function density(x,y){const dx=x-BG_C[0],dy=y-BG_C[1];return Math.exp(-(dx*dx+dy*dy)/(2*BG_S*BG_S));}
  const N_BG=350, N_SIG=12
  const points=[]
  for(let i=0;i<N_BG;i++){const xy=sampleBg();points.push({x:xy[0],y:xy[1],mass:sampleBgMass(),isSig:false,d:density(xy[0],xy[1])});}
  for(let i=0;i<N_SIG;i++){const xy=sampleSig();points.push({x:xy[0],y:xy[1],mass:sampleSigMass(),isSig:true,d:density(xy[0],xy[1])});}
  const order = points.map((p,i)=>i).sort((a,b)=>points[a].d-points[b].d)
  const NBINS=24, MASS_MIN=100, MASS_MAX=500
  function binOf(m){return Math.min(NBINS-1,Math.floor((m-MASS_MIN)/(MASS_MAX-MASS_MIN)*NBINS));}
  const scatterSvg = document.getElementById('scatter-svg')
  const histSvg = document.getElementById('hist-svg')
  function makeEl(svg,tag,attrs){const el=document.createElementNS(NS,tag);for(const k in attrs)el.setAttribute(k,attrs[k]);svg.appendChild(el);return el;}
  makeEl(scatterSvg,'circle',{cx:BG_C[0],cy:BG_C[1],r:1.5*BG_S,fill:'none',stroke:'var(--color-border-secondary)','stroke-width':1,'stroke-dasharray':'4 4'})
  const ptEls = points.map(p=>makeEl(scatterSvg,'circle',{cx:p.x,cy:p.y,r:3,class:'scatter-pt',fill:'var(--color-text-tertiary)'}))
  const BAR_W = 580/NBINS
  const expected=[]
  for(let b=0;b<NBINS;b++){
    const left=MASS_MIN+b*(MASS_MAX-MASS_MIN)/NBINS
    const right=MASS_MIN+(b+1)*(MASS_MAX-MASS_MIN)/NBINS
    expected.push(Math.exp(-(left-MASS_MIN)/DECAY)-Math.exp(-(right-MASS_MIN)/DECAY))
  }
  const sumExpected = expected.reduce((a,b)=>a+b,0)
  for(let b=0;b<NBINS;b++) expected[b] = expected[b]/sumExpected*N_BG
  const maxExpected = Math.max.apply(null, expected)
  const scale = 150/maxExpected
  for(let b=0;b<NBINS;b++){
    const x=50+b*BAR_W
    const h=expected[b]*scale
    makeEl(histSvg,'rect',{x:x+1,y:180-h,width:BAR_W-2,height:h,fill:'var(--color-text-tertiary)'})
  }
  makeEl(histSvg,'line',{x1:50,y1:180,x2:630,y2:180,stroke:'var(--color-border-secondary)','stroke-width':0.5})
  ;[100,200,300,400,500].forEach(m=>{
    const x=50+(m-MASS_MIN)/(MASS_MAX-MASS_MIN)*580
    const t=makeEl(histSvg,'text',{x:x,y:198,'text-anchor':'middle',class:'ts',fill:'var(--color-text-tertiary)'})
    t.textContent=m
  })
  const axisLabel = makeEl(histSvg,'text',{x:340,y:206,'text-anchor':'middle',class:'ts',fill:'var(--color-text-tertiary)'})
  axisLabel.textContent='Invariant mass (GeV)'
  const flagBarEls=[]
  for(let b=0;b<NBINS;b++){
    const x=50+b*BAR_W
    const bar=makeEl(histSvg,'rect',{x:x+1,y:180,width:BAR_W-2,height:0,fill:'#D85A30',class:'hist-bar'})
    flagBarEls.push(bar)
  }
  function render(pct){
    const numFlag = Math.round(pct/100*points.length)
    const flagged = new Set(order.slice(0,numFlag))
    let sigCaught = 0
    const flagBins = new Array(NBINS).fill(0)
    points.forEach((p,i)=>{
      const isFlag = flagged.has(i)
      ptEls[i].setAttribute('fill', isFlag?'#D85A30':'var(--color-text-tertiary)')
      if(isFlag){ flagBins[binOf(p.mass)]++; if(p.isSig) sigCaught++; }
    })
    for(let b=0;b<NBINS;b++){
      const h = flagBins[b]*scale
      flagBarEls[b].setAttribute('y', 180-h)
      flagBarEls[b].setAttribute('height', h)
    }
    document.getElementById('flagged-count').textContent = numFlag
    document.getElementById('signal-caught').textContent = sigCaught+' / '+N_SIG
  }
  renderFn = render
  render(2)
})

function onThreshold(v){
  document.getElementById('thresh-label').textContent = v+'%'
  if(renderFn) renderFn(parseFloat(v))
}
</script>

<template>
  <div class="nf-anomaly">
    <h2 class="sr-only">Interactive illustration of density-based anomaly detection in collider physics. A smooth analytical background histogram is shown for the invariant mass spectrum, with flagged anomalous events overlaid on top in a contrasting color. Dragging the sensitivity slider changes which events are flagged, letting you watch a tiny hidden resonance bump emerge against the smooth background reference.</h2>
    <div style="display:flex;gap:16px;font-size:12px;color:var(--color-text-secondary);margin-bottom:8px">
      <span><span class="legend-dot" style="background:var(--color-text-tertiary)"></span>Normal</span>
      <span><span class="legend-dot" style="background:#D85A30"></span>Flagged as anomalous</span>
    </div>
    <div style="display:flex;gap:24px;align-items:flex-start">
      <div style="flex:1;min-width:0">
        <svg id="scatter-svg" width="100%" viewBox="0 0 680 280"></svg>
      </div>
      <div style="flex:1;min-width:0">
        <div style="font-size:12px;color:var(--color-text-secondary);margin-bottom:4px">Flagged events vs smooth background reference</div>
        <svg id="hist-svg" width="100%" viewBox="0 0 680 210"></svg>
      </div>
    </div>
    <div style="display:flex;align-items:center;gap:12px;margin-top:14px">
      <label style="font-size:13px;color:var(--color-text-secondary);white-space:nowrap">Sensitivity</label>
      <input type="range" id="thresh-slider" min="1" max="40" step="1" value="2" style="flex:1" @input="onThreshold($event.target.value)">
      <span id="thresh-label" style="min-width:34px;text-align:right;font-size:13px">2%</span>
    </div>
    <div style="display:flex;gap:12px;margin-top:12px">
      <div class="stat-card" style="flex:1"><div style="font-size:12px;color:var(--color-text-secondary)">Flagged events</div><div id="flagged-count" style="font-size:20px;font-weight:500">0</div></div>
      <div class="stat-card" style="flex:1"><div style="font-size:12px;color:var(--color-text-secondary)">True signal caught</div><div id="signal-caught" style="font-size:20px;font-weight:500">0 / 12</div></div>
    </div>
  </div>
</template>

<style scoped>
.nf-anomaly{
  --color-background-secondary:#262624;
  --color-text-secondary:#cfcdc4;
  --color-text-tertiary:#a3a199;
  --color-border-secondary:rgba(255,255,255,0.35);
  --border-radius-md:8px;
  font-family:inherit;
  width:100%;
  height:100%;
  box-sizing:border-box;
  display:flex;
  flex-direction:column;
  justify-content:center;
  padding:2rem 3rem;
}
.nf-anomaly .sr-only{position:absolute;width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0,0,0,0);white-space:nowrap;border:0;}
.nf-anomaly .ts{font-size:12px;}
.nf-anomaly .legend-dot{width:10px;height:10px;border-radius:50%;display:inline-block;margin-right:4px}
.nf-anomaly .stat-card{background:var(--color-background-secondary);border-radius:var(--border-radius-md);padding:.75rem 1rem}
.nf-anomaly .scatter-pt{transition:fill .2s}
.nf-anomaly .hist-bar{transition:height .2s,y .2s}
</style>
