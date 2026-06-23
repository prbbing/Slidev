---
theme: default
layout: cover
title: 碰撞中的智慧
author: 刘炳萱
colorSchema: dark
---

<img src="/collision_animation_1080p.gif" class="absolute inset-0 w-full h-full object-cover -z-10" />

# 碰撞中的智慧

粒子物理实验中的人工智能

刘炳萱

中山大学理学院

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  开始我们的讨论 <carbon:arrow-right />
</div>

<div class="abs-br m-6 text-xl">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="slidev-icon-btn">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
---

# 粒子物理实验

粒子物理研究基本粒子以及它们之间的相互作用，我们发展出的理论需要用实验来检验

- 📝 **第一性原理** - 有完备的理论支撑
- 🎨 **高可信模拟** - 可以生成高质量模拟数据
- 💾 **大样本数据** - 对撞机产生万亿量级真实数据
- 🤹 **合作组模式** - 大规模国际化工作模式（ATLAS实验有3000多名成员）

<div class="absolute bottom-10 inset-x-0 h-2/5 flex items-center gap-10 px-10">
  <img src="/SM_MUG.jpg" class="w-[40%] h-full rounded-lg object-cover ml-auto" />
  <div class="w-[40%] text-center">

  粒子物理标准模型的简化版公式可以写在一个马克杯上
  </div>
</div>

---
layout: default
---

# 对撞机和探测器
大型强子对撞机（LHC）是人类历史上建造的最复杂实验设备，它周长为27公里，在瑞士-法国边境100米地下。两束质子束流加速到6.8TeV能量之后在固定位置对撞（对撞点）。探测器便安装在对撞点，通常为圆筒状结构一层层把对撞点包裹起来，对撞后产生的例子就会被探测到。右图是ATLAS探测器记录的一个事例

<div class="flex items-start gap-10 h-[80%] p-10">
  <div class="w-1/2 h-full">
    <LHCRing />
  </div>
  <div class="w-1/2 h-full">
    <img src="/multib_event.png" class="w-full h-full object-contain" />
  </div>
</div>

---
layout: default
---

# 大数据挑战
LHC产生的数据量级极大，是社交网络、流媒体平台等产生数据的千倍之多。但是受限于存储能力和算力，我们只能记录百万分之一，但这也是一个很大的数据量了，跟部分知名流媒体与社交网络数据量处在同一量级

<div class="flex items-center justify-center gap-5 p-5">
  <img src="/bigdata.png" class="w-[90%] rounded-lg object-cover" />
</div>

---
layout: default
---

# 喷注分类（Jet Classification）
粒子物理实验中的经典应用场景，我们需要鉴别事例中两种不同的喷注，大家可以简单理解为分辨两种不同的图片

<div class="flex h-full items-start gap-10 p">
  <div class="w-[80%] h-[80%]">
    <JetDecayCones />
  </div>
  <div class="w-[54%] space-y-4">

  ## 两种不同的喷注
  - <p class="text-gray-500">左边的轻味喷注（Light-flavour Let）中径迹都来自主顶点（Primary Vertex, PV）</p>
  - <p class="text-gray-500">右边的重味喷注（Heavy-flavour Jet）中的径迹可能给来自于次级顶点（Secondary Vertex, SV）</p>

  <div class="text-center">

  ## 你的第一反应可能是：这太简单了呀！

  </div>
  </div>
</div>

---
layout: default
---

# 嘈杂的环境
强子对撞机中会出现大量的粒子，整个事例非常嘈杂。同时探测器也会出现噪音，所以我们是要在极其复杂的环境中来鉴别两种喷注。而且真实情况下两者区别没这么明显，示意动画中做了夸张处理

<div style="height: 75%">
  <JetEventDisplay />
</div>

---
layout: default
---

# 算法演进

喷注标记因为之前提到的复杂性一直是机器学习的试验田，早在90年代就有人曾经提出使用神经网络来实现喷注标记

<div class="flex items-start gap-10 p-1">
  <img src="/jettagging_1991.png" class="w-[70%] rounded-lg object-cover" />
  <div class="w-[70%]">

  ## 前沿学科
  <p class="text-gray-500"> 作为最基础的实验学科我们一直以来都在技术前沿，这些早期的探索因为计算条件限制没有大规模应用

  我们一直是以解决问题为导向，如果一个简单的算法就能满足需求，我们不会因为有了新技术而去使用新技术

  喷注标记算法效能一直在随着算法的发展不断提升</p>
  </div>
</div>

<p class="text-gray-500"> 在喷注标注方向我们完整经历了近十年来机器学技术的发展，衍生出契合粒子物理研究特点的算法、软件生态环境。线面我们可以简单梳理一下喷注标记算法的发展脉络 </p>

---
layout: full
---

<JetTaggerEvolution />

---
layout: default
---

# Transformer

Transformer在粒子物理领域也展现出强大潜能，是目前各类问题解决方案的首选架构。我们的整体算法设计逻辑也由以特征工程为主转变为端到端，直接使用底层信息作为输入。探测器中获得的基本数据单元（径迹）可以看做词元，它们组合成序列（喷注）

<div style="height: 75%">
  <JetAsSequence />
</div>

---
layout: full
---

<JetTaggerInference />

---
layout: default
---

# 归纳偏置（Inductive Bias）

我们欣慰地发现，基于长期以来积累的物理知识与实验经验设置的归纳偏执可以有效地提升训练效果


<div class="flex h-full items-start gap-10 p">
  <div class="w-[80%] h-[80%]">
    <JetTaskHeads />
  </div>
  <div class="w-[54%]">

  
  - <p class="text-gray-500">除了分类主网络，我们还添加了两个子网络</p>
  - <p class="text-gray-500">它们基于我们的物理领域知识，即来自不同喷注的径迹和顶点是不一样的</p>
  - <p class="text-gray-500">通过模拟数据我们能够得到径迹和顶点的标签</p>
  

  ## 联合损失函数

  - <span class="text-gray-500"> $L = L_{\mathrm{jet}} + \alpha L_{\mathrm{track}} + \beta L_{\mathrm{vertex}}$ </span>
  - <p class="text-gray-500">特征工程以子网络的形式重生 </p>
  - <p class="text-gray-500">这也是体现各学科特色的环节 </p>

  </div>
</div>

---
layout: default
---

# 影响力深远

<div class="flex items-center justify-center gap-10 p-10">
  <img src="/GN2_NC.png" class="w-[80%] rounded-lg object-cover" />
  <div class="w-[80%]">

  ## 粒子物理
  <p class="text-gray-500">算法效能有了数倍提升并且架构具有很高的可延展性，能极大提升物理研究的潜力</p>

  ## 人工智能
  <p class="text-gray-500">Transformer架构与粒子物理领域知识的结合，在最极端实验室环境中证明了它的效能</p>
  - <p class="text-gray-500">大型强子对撞机复现了宇宙大爆炸之后很短一段时间的状态</p>

  </div>
</div>

---
layout: full
background: '#000000'
---
<img src="/The_History_of_the_Universe.jpg" class="w-full h-full object-fill" />

---
layout: default
---

# 异常检测

物理学史上的很多突破源于实验中观测到的异常数据。例如，一百多年前伦琴无意中在一张照片中发现了X-射线的踪迹。LHC对撞产生了大量数据，其中也极有可能纯在异常现象，只不过在这种场景下我们不能再像以前那样一张一张看照片了

<div class="flex flex-col h-full">
<div class="flex items-start gap-10 flex-1 p-10">
  <img src="/buble_chamber_image2.jpg" class="w-1/2 h-[70%] object-contain" />
  <img src="/buble_chamber_image3.jpg" class="w-1/2 h-[70%] object-contain" />
</div>
</div>
---
layout: default
---

# 异常检测

将数据转换到一个新的空间，在这其中异常数据将会和普通数据区分开来。这个方法也经历了从线性模型（例如k-means聚类）到现代架构（变分自编码器和标准化流等）的发展。

<div style="height: 75%">
  <AnomalyDetector />
</div>

---
layout: default
---

# 直接利用潜空间

我们习惯了利用物理上清晰、100%可解释的变量来开展研究，机器学习技术给我们带来的最大冲击来自于人类不能直观理解的高维潜空间。虽然这让人很不习惯，在研究中我们也开始尝试直接使用潜空间表针

<div class="flex h-full items-start gap-10 p">
  <div class="w-[80%] h-[80%]">
    <ParticleFlowVAE />
  </div>
  <div class="w-[54%] space-y-4">

  ## 有监督与无监督的结合

  - <p class="text-gray-500">首先通过一个有监督的基于DeepSet架构的分类器得到潜空间表征（中间层），利用模拟数据训练</p>
  - <p class="text-gray-500">再将这个潜空间表征应用到数据上作为一个变分自编码器的输入，直接利用真实数据训练</p>

  ## 目的性补缺

  <p class="text-gray-500"> 有监督算法中使用的标签可能带来一定偏差，直接利用学到的潜空间进行一个无监督训练可以有目的性地查漏补缺</p>

  </div>
</div>

---
layout: default
---

# 学生培养 

我们可能是最后一批没有经受过AI洗礼的粒子物理博士生。现在AI工具遍地开花，极大改变了学生学习模式以及学生培养模式。在这个时代下粒子物理专业的学生有什么与众不同的地方？他们能给业界带来什么？

- ✅ **解决问题为导向** - 每一个合格研究生都成功解决了一些问题（也包括一部分参与了科研训练的本科生）
- 📝 **第一性原理支持** - 很多问题有着唯一客观解释，学生在使用AI工具的过程中锻炼出较好的辨别能力
- 💾 **全闭环科研训练** - 我们也在与时俱进，给学生设置一些可以应用机器学习或者人工智能技术的课题，让学生从问题调研开始，实现数据获取、数据处理、模型训练、模型部署与效果研究的完整训练

<div class="flex flex-col h-full">

<div class="flex items-start gap-10 flex-1 p-10">
  <img src="/dijet_paper.png" class="w-1/2 h-[50%] object-contain" />
  <img src="/svj_paper.png" class="w-1/2 h-[50%] object-contain" />
</div>
</div>
