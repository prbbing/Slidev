<template>
  <div class="tf-flow">
    <h2 class="sr-only">Animated overlay on a transformer-based jet-tagging architecture diagram, showing data flowing from track and jet inputs through the track initialiser, into the transformer's self-attention block, and then fanning out into three task-specific heads: jet flavour, track origin, and vertex grouping.</h2>
    <div class="stage">
      <img src="/fig_01.png" alt="Transformer-based jet tagging architecture" />
      <div class="flow-dot flow-main"></div>
      <div class="flow-ring"></div>
      <div class="flow-dot flow-top"></div>
      <div class="flow-dot flow-mid"></div>
      <div class="flow-dot flow-bot"></div>
    </div>
  </div>
</template>

<script setup>
</script>

<style scoped>
.tf-flow{ width:100%; }
.stage{ position:relative; width:100%; }
.stage img{ width:100%; display:block; border-radius:8px; }
.sr-only{position:absolute;width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0,0,0,0);white-space:nowrap;border:0;}
.flow-dot{
  width:1.1%; aspect-ratio:1; min-width:10px; border-radius:50%;
  position:absolute; transform:translate(-50%,-50%);
  background:#00BFFF; border:2px solid #fff; box-sizing:border-box;
}
.flow-ring{
  width:7%; aspect-ratio:1; border-radius:50%; position:absolute;
  left:52%; top:44%; border:3px solid #00BFFF; transform:translate(-50%,-50%);
  animation:pulse-ring 8s linear infinite;
}
.flow-main{ animation:flow-main 8s linear infinite; }
.flow-top{ animation:flow-top 8s linear infinite; }
.flow-mid{ animation:flow-mid 8s linear infinite; }
.flow-bot{ animation:flow-bot 8s linear infinite; }

@keyframes flow-main{
  0%,100%{ left:12%; top:81%; opacity:0; }
  3%{ opacity:1; }
  12%{ left:22%; top:81%; }
  22%{ left:26%; top:60%; }
  30%{ left:29%; top:45%; }
  45%{ left:40%; top:45%; }
  60%{ left:52%; top:44%; }
  75%{ left:67%; top:44%; }
  82%{ left:72%; top:44%; opacity:1; }
  86%{ opacity:0; }
}
@keyframes pulse-ring{
  0%,55%,100%{ opacity:0; transform:translate(-50%,-50%) scale(.4); }
  62%{ opacity:.7; transform:translate(-50%,-50%) scale(1); }
  72%{ opacity:0; transform:translate(-50%,-50%) scale(1.6); }
}
@keyframes flow-top{
  0%,82%,100%{ left:72%; top:44%; opacity:0; }
  85%{ opacity:1; }
  90%{ left:78%; top:22%; }
  95%{ left:87%; top:22%; }
  98%{ left:95%; top:22%; opacity:1; }
}
@keyframes flow-mid{
  0%,82%,100%{ left:72%; top:44%; opacity:0; }
  85%{ opacity:1; }
  90%{ left:78%; top:44%; }
  95%{ left:87%; top:44%; }
  98%{ left:95%; top:44%; opacity:1; }
}
@keyframes flow-bot{
  0%,82%,100%{ left:72%; top:44%; opacity:0; }
  85%{ opacity:1; }
  90%{ left:78%; top:67%; }
  95%{ left:87%; top:67%; }
  98%{ left:95%; top:67%; opacity:1; }
}
</style>
