---
theme: default
title: Skein：为 AI 时代打造的嵌入式知识数据库
author: Weizhen Wang
aspectRatio: 16/9
canvasWidth: 980
fonts:
  sans: Inter
  mono: Space Grotesk
  local: Noto Sans SC
  provider: google
defaults:
  transition: fade
drawings:
  enabled: true
  persist: false
htmlAttrs:
  lang: zh
---

<div class="cover-hero grid h-full min-h-0 w-full grid-cols-1 items-center gap-y-8 md:grid-cols-[minmax(0,25rem)_minmax(0,1fr)] md:gap-x-10 lg:gap-x-14">
<div class="cover-head flex min-h-0 min-w-0 flex-col justify-center pr-1 md:pr-4">

<div class="sec cover-head__kicker">Nowledge Labs · PyConf 2026</div>

# Skein

## 为 AI 时代打造的嵌入式知识数据库

<div class="cover-head__lede">
一个 AI Agent 记忆系统的存储引擎重构故事：<br/>
把图数据库、向量索引、关系型存储合并成一个嵌入式引擎，会发生什么。
</div>

<div class="cover-foot mt-7 md:mt-8">
  <div class="flex flex-wrap items-center gap-x-3 gap-y-2" aria-label="Nowledge Labs · Nowledge Mem">
    <img src="./images/nowledge-labs-icon.png" alt="Nowledge Labs" class="cover-foot__logo cover-foot__logo--labs" />
    <span class="cover-foot__rule" aria-hidden="true"></span>
    <img src="./images/nowledge-mem-logo.webp" alt="Nowledge Mem" class="cover-foot__logo cover-foot__logo--mem" />
  </div>
  <div class="mt-3 flex flex-col gap-0.5 leading-snug">
    <span class="c1 text-[0.95rem] font-semibold tracking-tight">Weizhen Wang</span>
    <span class="c3 text-[0.8125rem] tracking-[0.02em]">@hawkingrei · Nowledge Labs · Nowledge Mem</span>
  </div>
</div>

</div>

<div class="cover-shot-column flex min-h-0 min-w-0 items-center justify-center md:justify-end">
<div class="merge-graphic">

<div class="merge-row">
  <div class="merge-node merge-node--sky"><ph-graph class="merge-node__icon" /><div class="merge-node__label">Kuzu /<br/>Ladybug</div><div class="merge-node__role">图存储</div></div>
  <div class="merge-node merge-node--teal"><ph-magnifying-glass class="merge-node__icon" /><div class="merge-node__label">LanceDB</div><div class="merge-node__role">向量 · 全文</div></div>
  <div class="merge-node merge-node--rose"><ph-table class="merge-node__icon" /><div class="merge-node__label">SQLite</div><div class="merge-node__role">关系型内容</div></div>
</div>

<svg class="merge-lines" viewBox="0 0 300 68" width="100%" height="68" preserveAspectRatio="none" aria-hidden="true">
  <path d="M50,0 C50,38 100,52 150,64" fill="none" stroke="var(--sky)" stroke-width="2.6" stroke-linecap="round" opacity="0.8" />
  <path d="M150,0 L150,64" fill="none" stroke="var(--teal)" stroke-width="2.6" stroke-linecap="round" opacity="0.8" />
  <path d="M250,0 C250,38 200,52 150,64" fill="none" stroke="var(--rose)" stroke-width="2.6" stroke-linecap="round" opacity="0.8" />
</svg>

<div class="merge-node merge-node--unified"><ph-git-merge class="merge-node__icon" /><div class="merge-node__label">Skein</div><div class="merge-node__role">一个嵌入式引擎，三份数据合一</div></div>

</div>
</div>
</div>

<!--
开场：
- 不是一个"我们发布了新产品"的分享，是一个存储工程决策的分享。
- 开场可以直接问观众："如果你的 AI Agent 记忆，同时活在三个数据库里——一个图库、一个向量库、一个关系库——你会怎么保证它们一致？"
- 停顿。然后说："我们也没有很好的答案。所以我们在造第四个东西，把前三个合并掉。这就是今天要讲的 Skein。"
-->

---

<div class="sec mb-4">Overview</div>

# Overview

<div class="mt-6 grid grid-cols-5 gap-5">

<div>
  <div class="c4 text-xs font-mono mb-2">01</div>
  <div class="c1 text-sm font-semibold mb-2">Why</div>
  <div class="c3 text-xs leading-relaxed">三个数据库，一份记忆，谁来保证一致</div>
</div>

<div>
  <div class="c4 text-xs font-mono mb-2">02</div>
  <div class="c1 text-sm font-semibold mb-2">What</div>
  <div class="c3 text-xs leading-relaxed">Skein 是什么：嵌入式、统一引擎、面向 AI 的查询</div>
</div>

<div>
  <div class="c4 text-xs font-mono mb-2">03</div>
  <div class="c1 text-sm font-semibold mb-2">How</div>
  <div class="c3 text-xs leading-relaxed">先建模再编码：64 个 TLA+ 规格与诚实的边界</div>
</div>

<div>
  <div class="c4 text-xs font-mono mb-2">04</div>
  <div class="c1 text-sm font-semibold mb-2">Status</div>
  <div class="c3 text-xs leading-relaxed">现状、两个真实的教训，和接下来两件事</div>
</div>

<div>
  <div class="c4 text-xs font-mono mb-2">05</div>
  <div class="c1 text-sm font-semibold mb-2">So What</div>
  <div class="c3 text-xs leading-relaxed">这跟一个 Python 大会有什么关系</div>
</div>

</div>

<!--
30 秒过场。手势扫五列："今天分五段：先看问题，再看 Skein 是什么，然后看我们怎么证明它是对的，现状诚实地讲一遍，最后聊聊为什么这跟你们有关系。"
-->

---
layout: center
class: deck-part-hero
---

<div class="text-center deck-section-hero">

<div class="progress-bar mb-8 justify-center"><span class="active">01 Why</span><span class="dot">·</span><span>02 What</span><span class="dot">·</span><span>03 How</span><span class="dot">·</span><span>04 Status</span><span class="dot">·</span><span>05 So What</span></div>

<div class="c4 text-sm tracking-widest uppercase mb-4">Part 1</div>

# 为什么要重新造一个存储引擎

<div class="c3 mt-4 text-lg">
一个 AI Agent 的记忆，长在三个数据库里
</div>

</div>

<!--
过渡：从上一页的 Overview 直接进这页。"先说清楚我们为什么要造轮子——因为我们已经有三个轮子了，它们不咬合。"
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span class="active">01 Why</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 一份记忆，三份存储

<div class="deck-split">

<div>
<div v-click class="deck-challenge-lede c2 text-base leading-loose">
Nowledge Mem 的每一条记忆，同时要满足三种访问模式：<br/>
它跟别的记忆的<strong class="c1">关系</strong>，它的<strong class="c1">语义相似度</strong>，还有它的<strong class="c1">原始内容</strong>。
</div>

<div v-click class="mt-4">
  <div class="c1 font-semibold mb-2">当时没有合适的一体化嵌入式引擎，于是我们用了三个专门的数据库</div>
  <div class="c2 text-sm leading-relaxed">
    <strong class="c1">Kuzu / Ladybug</strong>（图）· <strong class="c1">LanceDB</strong>（向量 + 全文）· <strong class="c1">SQLite</strong>（关系型内容）<br/>
    <span class="c3">每一个都是各自领域里成熟的选择。我们没有找到能在本地同时统一图遍历、向量/全文检索和事务性内容存储的引擎。</span>
  </div>
</div>

<div v-click class="mt-3 c2 text-sm leading-relaxed">
  <strong class="c1">这不是任何一个数据库不够好</strong>；难点在于一条记忆要被拆成三份状态，再由应用层把它们重新拼成一个整体。
</div>
</div>

<div v-click class="flex items-center">
<div class="stores-row" style="flex-direction: column; align-items: center; gap: 0.5rem;">
  <div class="store-card"><div class="store-card__label">Kuzu / Ladybug</div><div class="store-card__role">图存储 · 关系</div></div>
  <div class="store-arrow">＋</div>
  <div class="store-card"><div class="store-card__label">LanceDB</div><div class="store-card__role">向量 · 全文索引</div></div>
  <div class="store-arrow">＋</div>
  <div class="store-card"><div class="store-card__label">SQLite</div><div class="store-card__role">关系型内容存储</div></div>
  <div class="store-arrow cr font-semibold">= 谁来保证一致？</div>
</div>
</div>

</div>

</div>

<!--
讲法提示：
- "问题不在任何一个数据库本身，Kuzu、LanceDB、SQLite 单独看都很好。我们当时没有找到一款适合本地嵌入、同时覆盖三种访问模式的一体化引擎。"
- 所以我们把每种模式交给最擅长它的存储；真正的代价是同一条记忆被拆成三份状态，系统必须把它们重新拼起来。
- 过渡到下一页："最常见的问题是：那为什么不干脆只用 SQLite？"
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span class="active">01 Why</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 为什么不是直接用 SQLite？

<div class="deck-challenge-lede c2 text-sm leading-relaxed mt-3">
<strong class="c1">嵌入是部署形态，不是性能定位。</strong>SQLite 和 Skein 都以库的形式进入应用进程：无需独立服务。AI 主机会成为多个 Agent 共用的计算节点，需要把查询、写入和维护放进同一套资源治理。
</div>

<div class="db-capability-grid mt-5">
  <div class="db-capability db-capability--sqlite">
    <div class="db-capability__name">SQLite</div>
    <div class="db-capability__role">内容与事务</div>
    <div class="db-capability__strength">✓ 关系行、ACID、嵌入式部署<br/>✓ FTS5 做词法全文检索</div>
    <div class="db-capability__limit">图要自己建节点/边表再写 recursive CTE；ANN 向量检索要接扩展。多个 Agent 同时检索、写入和维护时，跨模态查询与资源调度仍要由应用层组合。</div>
  </div>
  <div class="db-capability db-capability--graph">
    <div class="db-capability__name">Kuzu / Ladybug</div>
    <div class="db-capability__role">关系与多跳遍历</div>
    <div class="db-capability__strength">✓ 图身份、关系、Cypher<br/>✓ 多跳路径与图算子</div>
    <div class="db-capability__limit">不适合承载长内容；在 Mem 里，向量/全文仍需要独立的搜索投影。</div>
  </div>
  <div class="db-capability db-capability--search">
    <div class="db-capability__name">LanceDB</div>
    <div class="db-capability__role">向量与全文检索</div>
    <div class="db-capability__strength">✓ 向量召回、BM25 / FTS<br/>✓ 快速、可重建的检索投影</div>
    <div class="db-capability__limit">它不是事实源：删除或损坏后要从图和内容存储重建，不能独自拥有一条记忆。</div>
  </div>
</div>

<div v-click class="db-unification mt-5">
  <span>图关系</span><b>＋</b><span>语义/全文检索</span><b>＋</b><span>原始内容</span>
  <strong>一份数据 · 一个 catalog · 一个 WAL · 一个查询计划</strong>
  <em>多 Agent 共用 · 用户查询优先 · 维护任务受控</em>
</div>

</div>

<!--
讲法提示：
- 这里的“嵌入式”是数据库作为库进入应用进程，不是面向低配嵌入式设备。SQLite 和 Skein 都可这样部署，SQLite 的 FTS5 也很成熟。这里不是“SQLite 不行”。
- AI 主机的负载不同：同一台机器会成为多个 Agent 共用的计算节点，查询、写入和后台投影/维护必须争取同一份 CPU、内存和 I/O 预算。
- SQLite 可以用边表和 recursive CTE 做图，也可以通过扩展获得 ANN；代价是我们仍要把图、向量、全文和它们的生命周期组合起来，并在应用层安排这些工作。
- Kuzu/Ladybug 和 LanceDB 同理：前者是关系事实源，后者是可重建检索投影。三者各自正确，但没有一个独自拥有整条 Mem 访问路径。
- Skein 追求的是最后两行的统一边界：同一份 catalog、WAL、查询计划，以及多 Agent 工作负载的前后台资源治理；这不是宣称替代所有通用数据库。
- 过渡到下一页："这就是组合以后，真正出现的问题。"

[Sources]
- SQLite, "Appropriate Uses For SQLite," 2025-05-31: https://www.sqlite.org/whentouse.html
- SQLite, "Write-Ahead Logging," accessed 2026-09-03: https://www.sqlite.org/wal.html
- Skein, "Architecture," local source: skein/docs/ARCHITECTURE.md
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span class="active">01 Why</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 问题不在选择，而在边界

<div class="deck-challenge-lede c2 text-base leading-loose mb-5">
每一个系统都适合自己的访问模式；难点是同一条记忆被投影成三份<strong class="c1">可以独立失败的状态</strong>。
</div>

<div class="grid grid-cols-2 gap-x-10 gap-y-6 text-sm leading-relaxed">
  <div v-click>
    <div class="c1 font-semibold mb-1"><span class="num">01</span> 一致性不是原子的</div>
    <div class="c2">一次写入拆成三次提交；没有跨库事务，崩溃后的补偿与对账留给应用。</div>
  </div>
  <div v-click>
    <div class="c1 font-semibold mb-1"><span class="num">02</span> 索引会落后于内容</div>
    <div class="c2">图和向量都是内容的投影；异步更新、失败重试和重建窗口都会制造短暂不一致。</div>
  </div>
  <div v-click>
    <div class="c1 font-semibold mb-1"><span class="num">03</span> 读路径要跨库拼接</div>
    <div class="c2">图或向量查询先返回 ID，再回内容存储 hydration；额外的 lookup 与序列化也会放大尾延迟。</div>
  </div>
  <div v-click>
    <div class="c1 font-semibold mb-1"><span class="num">04</span> 演进与恢复是三份工作</div>
    <div class="c2">Schema、WAL、备份和可观测性各走一套；定位一次问题，要拼三份证据。</div>
  </div>
</div>

<div v-click class="callout mt-6">
  Skein 的目标不是否定这些专用系统；而是把一份记忆的事务、WAL、catalog 和查询执行收回到同一个嵌入式引擎：同时追求正确性与性能。
</div>

</div>

<!--
讲法提示：
- 这四点是同一个问题的四个表面：同一份用户状态被拆到多个独立的故障域，而应用层成了唯一的协调者。
- "索引会落后于内容"不是说索引异步一定错误；而是在失败、重试和重建期间，应用必须定义并维持一致性语义。
- Skein 不是要替所有专用数据库做通用替代品。它聚焦于 Mem 需要的本地嵌入式统一数据模型和执行路径；少掉跨库 lookup、序列化和 hydration，也是性能目标，不只是正确性的附带收益。
- 过渡到下一页："所以，AI 时代的记忆到底需要什么样的存储？"
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span class="active">01 Why</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# AI 时代的记忆，需要什么样的存储？

<div class="mt-8 grid grid-cols-2 gap-x-10 gap-y-6">

<div v-click class="flex gap-3">
  <div class="num">1</div>
  <div><div class="c1 font-semibold">本地优先</div><div class="c2 text-sm mt-1 leading-relaxed">Agent 记忆必须离线可用，不能依赖一次网络往返</div></div>
</div>

<div v-click class="flex gap-3">
  <div class="num">2</div>
  <div><div class="c1 font-semibold">图状</div><div class="c2 text-sm mt-1 leading-relaxed">知识天然是实体 + 关系，不是拍平的表</div></div>
</div>

<div v-click class="flex gap-3">
  <div class="num">3</div>
  <div><div class="c1 font-semibold">检索是一等公民</div><div class="c2 text-sm mt-1 leading-relaxed">语义搜索不该是拼贴在图库外面的一个索引</div></div>
</div>

<div v-click class="flex gap-3">
  <div class="num">4</div>
  <div><div class="c1 font-semibold">崩溃安全</div><div class="c2 text-sm mt-1 leading-relaxed">写入和一致性不能是"大概率正确"</div></div>
</div>

<div v-click class="flex gap-3 col-span-2 justify-center">
  <div class="num">5</div>
  <div><div class="c1 font-semibold">可嵌入</div><div class="c2 text-sm mt-1 leading-relaxed">像 SQLite 一样链接进程，而不是再运维一个数据库服务</div></div>
</div>

</div>

</div>

<!--
念完五点后停顿一下："这五条里，最后一条'可嵌入'，是很多人容易忽略的一条。我们先讲讲为什么它这么重要，再看 Skein 怎么把前四条也做进去。"
-->

---
layout: center
class: deck-part-hero
---

<div class="text-center deck-section-hero">

<div class="progress-bar mb-8 justify-center"><span>01 Why</span><span class="dot">·</span><span class="active">02 What</span><span class="dot">·</span><span>03 How</span><span class="dot">·</span><span>04 Status</span><span class="dot">·</span><span>05 So What</span></div>

<div class="c4 text-sm tracking-widest uppercase mb-4">Part 2</div>

# Skein 是什么

<div class="c3 mt-4 text-lg">
嵌入式 · 统一引擎 · 面向 AI 的查询
</div>

</div>

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span class="active">02 What</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 电梯演讲

<div class="mt-6 deck-closing-quote" style="max-width: 44rem; margin: 0 auto;">
"Skein 是一个嵌入式 Rust 数据库引擎，目标是成为 Nowledge Mem 的<strong class="c1">唯一</strong>本地存储层——<br/>
用一个 WAL、一个优化器、一个进程，取代 Kuzu + LanceDB + SQLite 三件套。"
</div>

<div v-click class="mt-6 text-center c2 text-base">
定位：<strong class="c1">SQLite，但是为 AI Agent 的图状记忆而生</strong>
</div>

<div v-click class="mt-6">
<div class="merge-graphic merge-graphic--lg mx-auto">

<div class="merge-row">
  <div class="merge-node merge-node--sky"><ph-graph class="merge-node__icon" /><div class="merge-node__label">Kuzu /<br/>Ladybug</div><div class="merge-node__role">图存储</div></div>
  <div class="merge-node merge-node--teal"><ph-magnifying-glass class="merge-node__icon" /><div class="merge-node__label">LanceDB</div><div class="merge-node__role">向量 · 全文</div></div>
  <div class="merge-node merge-node--rose"><ph-table class="merge-node__icon" /><div class="merge-node__label">SQLite</div><div class="merge-node__role">关系型内容</div></div>
</div>

<svg class="merge-lines" viewBox="0 0 300 68" width="100%" height="68" preserveAspectRatio="none" aria-hidden="true">
  <path d="M50,0 C50,38 100,52 150,64" fill="none" stroke="var(--sky)" stroke-width="2.6" stroke-linecap="round" opacity="0.8" />
  <path d="M150,0 L150,64" fill="none" stroke="var(--teal)" stroke-width="2.6" stroke-linecap="round" opacity="0.8" />
  <path d="M250,0 C250,38 200,52 150,64" fill="none" stroke="var(--rose)" stroke-width="2.6" stroke-linecap="round" opacity="0.8" />
</svg>

<div class="merge-node merge-node--unified"><ph-git-merge class="merge-node__icon" /><div class="merge-node__label">Skein</div><div class="merge-node__role">一个嵌入式引擎</div></div>

</div>
</div>

</div>

<!--
- 电梯演讲那句话，读两遍，第一遍正常速度，第二遍放慢，强调"唯一"两个字。
- 图示扫一遍："三个变一个，不是删掉功能，是把边界收进一个进程里。"
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span class="active">02 What</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 嵌入式优先，也可服务化

<div class="deck-split">

<div v-click>
  <div class="c1 font-semibold mb-2">AI Local：像 SQLite 一样，链接进程</div>
  <div class="c2 text-sm leading-relaxed">
    Agent 与数据库在同一进程：没有网络往返，没有独立部署，也不用再运维一个数据库服务。<br/>
    <span class="c3">这正是本地优先 AI Agent 需要的低延迟、离线可用形态。</span>
  </div>
</div>

<div v-click>
  <div class="c1 font-semibold mb-2">服务化：由宿主掌握调度权</div>
  <div class="c2 text-sm leading-relaxed">
    多个 Agent 或设备接入时，宿主把同一套 database facade 放在服务 API 后，并拥有业务优先级的全局视图。<br/>
    <span class="c3">按业务安排统计信息刷新、索引/投影维护与后台任务；把 CPU、内存和 I/O 留给用户正在等待的 Agent 查询。</span>
  </div>
</div>

</div>

<div v-click class="mt-6 callout">
<strong class="c1">一个内核，两种部署形态</strong>：本地直接嵌入；需要共享或统一治理时由宿主服务化。<br/>
<span class="c3">服务化的价值不只是远程访问，而是把数据库工作纳入业务调度；catalog、WAL、事务与查询计划仍然只有一份。</span>
</div>

</div>

<!--
- "AI Local 不是拒绝服务端，而是先保证本地路径没有服务依赖。需要共享或统一治理时，把同一个内核交给宿主服务化，不必复制一套数据库。"
- 服务化不只是开一个网络入口。宿主知道哪些是用户正在等的 Agent 查询，哪些是可以延后的统计信息刷新（例如未来的 auto analyze）、索引/投影维护和后台任务；它能按业务优先级把 CPU、内存、I/O 预算和调度顺序分给它们。
- 这是 host-owned policy：Skein 提供同一份 catalog、WAL、事务、查询计划及 QoS 接口；宿主决定业务优先级和 worker 的实际运行时机。不要把它描述成已存在的独立 Skein server 产品。

[Sources]
- Skein, "Architecture," local source: skein/docs/ARCHITECTURE.md (QoS and background-maintenance boundaries)
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span class="active">02 What</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 一个引擎，两种查询语言

<div class="mt-4 c2 text-sm">Cypher 和 PostgreSQL SQL / SQL-PGQ，只在"绑定"之后汇合——之后是同一套算子、同一个优化器、同一个存储引擎。</div>

<div class="query-flow mt-6" aria-label="Cypher and PostgreSQL SQL converge into the shared query execution pipeline">
  <div class="query-flow__lane query-flow__lane--cypher">
    <div class="pipe-box pipe-box--surface">Cypher<span class="pipe-box__sub">图查询</span></div>
    <div class="pipe-arrow" aria-hidden="true">→</div>
    <div class="pipe-box">AST</div>
    <div class="pipe-arrow" aria-hidden="true">→</div>
    <div class="pipe-box">语义图<span class="pipe-box__sub">查询模型</span></div>
  </div>

  <div class="query-flow__join-arrow query-flow__join-arrow--down" aria-hidden="true">↓</div>

  <div class="query-flow__core">
    <div class="pipe-box pipe-box--core">共享逻辑计划</div>
    <div class="pipe-arrow" aria-hidden="true">→</div>
    <div class="pipe-box pipe-box--core">Cascades<span class="pipe-box__sub">优化器</span></div>
    <div class="pipe-arrow" aria-hidden="true">→</div>
    <div class="pipe-box pipe-box--core">物理计划</div>
    <div class="pipe-arrow" aria-hidden="true">→</div>
    <div class="pipe-box pipe-box--core">执行器</div>
    <div class="pipe-arrow" aria-hidden="true">→</div>
    <div class="pipe-box pipe-box--core">行</div>
  </div>

  <div class="query-flow__join-arrow query-flow__join-arrow--up" aria-hidden="true">↑</div>

  <div class="query-flow__lane query-flow__lane--sql">
    <div class="pipe-box pipe-box--surface">PostgreSQL SQL<span class="pipe-box__sub">+ SQL/PGQ</span></div>
    <div class="pipe-arrow" aria-hidden="true">→</div>
    <div class="pipe-box">语法 AST</div>
    <div class="pipe-arrow" aria-hidden="true">→</div>
    <div class="pipe-box">类型绑定</div>
  </div>
</div>

<div v-click class="mt-5 c2 text-sm leading-relaxed">
SQL 侧对标 PostgreSQL master（<code class="text-xs">3d00537f</code>）的方言，实现的是 <strong class="c1">ISO/IEC 9075-16 SQL/PGQ</strong>——<code class="text-xs">CREATE PROPERTY GRAPH</code> 和作为 <code class="text-xs">FROM</code> 子句成员的 <code class="text-xs">GRAPH_TABLE</code>。<br/>
<span class="c3">诚实的边界：这是"SQL 里内嵌图查询"，不是独立的 ISO/IEC 39075 GQL 实现——也从不会把 SQL/PGQ 语句翻译成 Cypher 文本再执行，两条语言从解析开始就落到同一套类型化表达式上。</span>
</div>

</div>

<!--
"两条路走进来，一条从 Cypher，一条从标准 SQL 加 GRAPH_TABLE 语法——绑定完之后，走的是完全一样的优化器和执行器。这意味着你可以用 SQL 写关系查询，用 Cypher 写图遍历，查的是同一份数据，同一套一致性保证。"
- "SQL 那侧不是我们自己发明的方言，是在实现一个正在成型的 ISO 标准——SQL/PGQ，PostgreSQL 自己的主干也还在跟进同一个标准。我们特意没有去碰独立 GQL 那个更大的标准，范围是清楚的。"

[Sources]
- skein/docs/specs/POSTGRES_SQL_PGQ_SPEC.md
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span class="active">02 What</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# Optimizer workflow

<div class="deck-challenge-lede c2 text-sm leading-relaxed mt-3">
The optimizer turns a shared logical plan into a physical plan, choosing how to execute the query while preserving its meaning.
</div>

<div class="pipe-row mt-6" aria-label="Logical plan, rule rewrites, candidate plans, cost comparison, physical plan">
  <div class="pipe-box">Logical plan<span class="pipe-box__sub">What to compute</span></div>
  <div class="pipe-arrow" aria-hidden="true">→</div>
  <div class="pipe-box pipe-box--core">Rule rewrites<span class="pipe-box__sub">Simplify the plan</span></div>
  <div class="pipe-arrow" aria-hidden="true">→</div>
  <div class="pipe-box pipe-box--core">Candidate plans<span class="pipe-box__sub">Explore alternatives</span></div>
  <div class="pipe-arrow" aria-hidden="true">→</div>
  <div class="pipe-box pipe-box--core">Cost comparison<span class="pipe-box__sub">Compare estimates</span></div>
  <div class="pipe-arrow" aria-hidden="true">→</div>
  <div class="pipe-box">Physical plan<span class="pipe-box__sub">How to execute</span></div>
</div>

<div class="mt-6 c2 text-sm leading-relaxed space-y-3">
  <div v-click><strong class="c1">Rewrite:</strong> apply rules that simplify the logical plan without changing the result.</div>
  <div v-click><strong class="c1">Explore and compare:</strong> consider join orders and data access paths, then compare their estimated costs.</div>
  <div v-click><strong class="c1">Select:</strong> produce a physical plan for the executor.</div>
</div>

</div>

<!--
- Cypher and SQL arrive at the same logical plan, which describes what the query should compute.
- Rules first simplify that plan while preserving the query's meaning.
- The optimizer explores execution alternatives, such as join orders and ways to read the data, and compares their estimated costs.
- The selected physical plan tells the executor how to run the query. Cost estimates guide this choice; they do not guarantee the fastest runtime.

[Sources]
- skein/docs/ARCHITECTURE.md (shared query pipeline)
- skein/crates/optimizer/src/stage.rs (rule rewrites)
- skein/crates/optimizer/src/relational_join.rs (join planning)
- skein/crates/optimizer/src/relational.rs (access-path selection)
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span class="active">02 What</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# Executor workflow

<div class="deck-challenge-lede c2 text-sm leading-relaxed mt-3">
The executor follows the physical plan: each operator reads data, performs one operation, and passes its output downstream.
</div>

<div class="mt-5 c3 text-sm">Example: a query that filters rows and selects columns</div>

<div class="pipe-row mt-3" aria-label="Example data flow: scan, filter, project, results">
  <div class="pipe-box">Scan<span class="pipe-box__sub">Read source data</span></div>
  <div class="pipe-arrow" aria-hidden="true">→</div>
  <div class="pipe-box pipe-box--core">Filter<span class="pipe-box__sub">Keep matching rows</span></div>
  <div class="pipe-arrow" aria-hidden="true">→</div>
  <div class="pipe-box pipe-box--core">Project<span class="pipe-box__sub">Select output columns</span></div>
  <div class="pipe-arrow" aria-hidden="true">→</div>
  <div class="pipe-box">Results<span class="pipe-box__sub">Return to the caller</span></div>
</div>

<div class="mt-6 c2 text-sm leading-relaxed space-y-3">
  <div v-click><strong class="c1">Connected operators:</strong> the physical plan determines which operations run and how they connect.</div>
  <div v-click><strong class="c1">Batch flow:</strong> streaming operators process bounded batches and pass them to the next operator.</div>
  <div v-click><strong class="c1">Blocking work:</strong> sorting and aggregation accumulate state before producing results, under memory limits.</div>
</div>

</div>

<!--
- The optimizer has chosen a physical plan. The executor now carries out its operations.
- In this example, a scan reads data, a filter keeps matching rows, and a projection selects the output columns.
- Streaming operators pass bounded batches downstream. A stop signal can end that flow when the consumer has enough results.
- Sorting and aggregation need intermediate state before they can produce their results. This breaks the streaming flow and requires memory management.

[Sources]
- skein/src/executor/batch.rs (physical-plan dispatch, filtering, projection)
- skein/crates/executor/src/pipeline.rs (batch emission and stop propagation)
- skein/crates/executor/src/blocking.rs (blocking execution context)
- skein/docs/ARCHITECTURE.md (execution memory limits)
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span class="active">02 What</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 架构地图

<div class="deck-challenge-lede c2 text-sm leading-relaxed mt-3">
Skein 不是一个巨型 crate。我们按<strong class="c1">职责与依赖方向</strong>拆分：把变化快的语言前端放在边缘，把稳定的查询与存储内核锁在中间。
</div>

<div class="crate-legend mt-3" aria-label="Module categories">
  <span class="crate-legend__item crate-legend__item--surface">语言前端</span>
  <span class="crate-legend__item crate-legend__item--kernel">共享内核</span>
  <span class="crate-legend__item crate-legend__item--extension">投影与运营能力</span>
</div>

<div class="crate-grid">
  <div class="crate-card crate-card--kernel"><div class="crate-card__name">core</div><div class="crate-card__role">错误、值、ID、catalog、schema 描述</div></div>
  <div class="crate-card crate-card--surface"><div class="crate-card__name">cypher</div><div class="crate-card__role">Cypher 词法 / 语法 / AST</div></div>
  <div class="crate-card crate-card--surface"><div class="crate-card__name">sql-syntax / sql</div><div class="crate-card__role">PostgreSQL SQL + SQL/PGQ 语义降解</div></div>
  <div class="crate-card crate-card--kernel"><div class="crate-card__name">plan</div><div class="crate-card__role">共享逻辑 / 物理 IR，指纹，explain</div></div>
  <div class="crate-card crate-card--kernel"><div class="crate-card__name">optimizer</div><div class="crate-card__role">Cascades memo / rules，图代价模型</div></div>
  <div class="crate-card crate-card--kernel"><div class="crate-card__name">storage</div><div class="crate-card__role">存储协议、MVCC、WAL、索引</div></div>
  <div class="crate-card crate-card--kernel"><div class="crate-card__name">executor</div><div class="crate-card__role">物理算子 / 查询执行</div></div>
  <div class="crate-card crate-card--extension"><div class="crate-card__name">analytics</div><div class="crate-card__role">不可变 CSR/CSC 投影，PageRank / Louvain</div></div>
  <div class="crate-card crate-card--extension"><div class="crate-card__name">qos</div><div class="crate-card__role">资源分级、后台准入、期望值调度</div></div>
  <div class="crate-card crate-card--extension"><div class="crate-card__name">vector-projection</div><div class="crate-card__role">向量投影 / ANN 候选集扫描</div></div>
  <div class="crate-card crate-card--extension"><div class="crate-card__name">evidence</div><div class="crate-card__role">发布身份、崩溃恢复证据合约</div></div>
  <div class="crate-card crate-card--extension"><div class="crate-card__name">telemetry / readiness</div><div class="crate-card__role">可观测性、就绪度、qualification / fuzz</div></div>
</div>

<div class="callout mt-4 text-sm">
  根 crate 只保持稳定的嵌入式 API；内部模块维持单向依赖。搜索、图分析、QoS 与运行证据拥有独立边界，部分能力可以按 feature 组合——<code class="text-xs">vector-search</code>、<code class="text-xs">graph-analytics</code>、<code class="text-xs">background-maintenance</code>，以及下一页要讲的 <code class="text-xs">full-text-search</code>，都是可以按需裁掉的默认 feature。
</div>

</div>

<!--
"不要逐个念。这张图不是 crate 清单，而是模块化设计：蓝色是会变化的语言前端，橙色是稳定的共享内核，绿色是可独立演进的投影和运营能力。"
- 根 crate 只提供稳定的嵌入式 facade；内部实现可以在不扩大用户 API 的前提下演进。
- 依赖方向保持无环：上层不会反过来依赖下层。这样既能按 feature 组合能力，也把形式化与测试的责任边界说清楚。
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span class="active">02 What</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 全文检索是内核能力，不是外挂

<div class="deck-challenge-lede c2 text-sm leading-relaxed mt-3">
第一部分说过"检索是一等公民"——这不只是口号：全文检索是 Skein 的<strong class="c1">默认 Cargo feature</strong>，跟图查询、向量检索长在同一个引擎里，不是绑一个外部搜索服务。
</div>

<div class="deck-split mt-4">

<div v-click>
  <div class="c1 font-semibold mb-2">可重建投影，不是权威数据</div>
  <div class="c2 text-sm leading-relaxed">
    BM25 全文索引和 ANN 向量索引、属性索引、成员过滤器一样，都是<strong class="c1">可重建投影</strong>。<br/>
    <span class="c3">这份索引损坏，绝不能让图上的权威数据变得不可恢复——这跟前面"LanceDB 是可重建投影"是同一条设计哲学，现在原生长在 Skein 里。</span>
  </div>
</div>

<div v-click>
  <div class="c1 font-semibold mb-2">独立的生产验收路径</div>
  <div class="c2 text-sm leading-relaxed">
    <code class="text-xs">qualification/production_search</code> 和 <code class="text-xs">graph_search</code> 是两个独立模块——全文/图检索的生产就绪度单独验收，不搭图存储验收的便车。
  </div>
</div>

</div>

</div>

<!--
"很多人第一反应是'图数据库 + 全文检索'要接一个 Elasticsearch 或者外部搜索服务。我们的做法不一样：全文检索是内核 feature，默认打开，跟图查询、向量检索共享同一套存储和一致性边界。"
- "索引本身是可重建投影这条原则，不是新发明——上一页讲的向量投影也是同一套原则。BM25 索引坏了，重建就好，图上的权威数据不受影响。"

[Sources]
- skein/docs/specs/EMBEDDED_RUNTIME_SPEC.md
- skein/crates/qualification/src/production_search/, graph_search.rs
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span class="active">02 What</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 面向 AI 的具体能力：GraphRAG 安全查询生成

<div class="deck-split">

<div>
<div v-click class="deck-challenge-lede c2 text-base leading-loose">
这可能是整个项目里最直接的"AI 时代"注脚——<br/>
Skein 里有一条路径，专门设计给 LLM 用来安全地查询图谱。
</div>

<div v-click class="mt-4">
  <div class="c1 font-semibold mb-2">GraphRagSchemaContext</div>
  <div class="c2 text-sm leading-relaxed">
    给 LLM 一个"目录级"的图 schema 视图：labels、关系类型、常见的 1-2 跳路径。<br/>
    <span class="c3">绝不包含 payload 本体，也绝不包含 embedding 向量。</span>
  </div>
</div>

<div v-click class="mt-3">
  <div class="c1 font-semibold mb-1">LLM 起草，Skein 校验</div>
  <div class="c2 text-sm leading-relaxed">
    LLM 据此起草参数化的、只读的 Cypher。<br/>
    执行前校验：<span class="pill text-xs">标识符合法性</span> <span class="pill text-xs">跳数上界</span> <span class="pill text-xs">查询指纹</span><br/>
    <span class="c3">指纹不是查询本身的哈希，是 LLM 当时看到的那份 schema 快照的哈希——schema 变了，查询字面没变也会被拒绝执行。</span>
  </div>
</div>
</div>

<div v-click class="flex items-center">
<div class="pipe-row" style="flex-direction: column; align-items: stretch; gap: 0.5rem;">
  <div class="pipe-box pipe-box--surface">LLM<span class="pipe-box__sub">读 schema 目录</span></div>
  <div class="pipe-arrow" style="text-align:center;">↓</div>
  <div class="pipe-box">起草参数化 Cypher</div>
  <div class="pipe-arrow" style="text-align:center;">↓</div>
  <div class="pipe-box pipe-box--core">Skein 校验<span class="pipe-box__sub">标识符 · 跳数 · 指纹</span></div>
  <div class="pipe-arrow" style="text-align:center;">↓</div>
  <div class="pipe-box">只读执行</div>
</div>
</div>

</div>

</div>

<!--
"数据库不再只是被动等 SQL 进来。这条路径是专门为'另一端是一个会犯错的 LLM'设计的——给它一个安全的、有边界的 schema 视图，让它写查询，然后在执行前把它当作不可信输入来校验。这跟传统 ORM 的思路完全不一样。"
-->

---
layout: center
class: deck-part-hero
---

<div class="text-center deck-section-hero">

<div class="progress-bar mb-8 justify-center"><span>01 Why</span><span class="dot">·</span><span>02 What</span><span class="dot">·</span><span class="active">03 How</span><span class="dot">·</span><span>04 Status</span><span class="dot">·</span><span>05 So What</span></div>

<div class="c4 text-sm tracking-widest uppercase mb-4">Part 3</div>

# 正确性，是设计出来的

<div class="c3 mt-4 text-lg">
先建模，再编码——以及诚实的验证边界
</div>

</div>

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span class="active">03 How</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 先建模，后编码

<div class="stat-row">
  <div class="stat"><div class="stat__num">64</div><div class="stat__label">TLA+ 规格<br/>覆盖几乎每一个子系统</div></div>
  <div class="stat"><div class="stat__num">0</div><div class="stat__label">跨库事务<br/>——三库并存时代的数字</div></div>
</div>

<div v-click class="mt-6 c2 text-sm leading-relaxed">
在写并发 / 崩溃恢复相关代码之前，先枚举状态、事件、迁移、非法迁移、所有权、过期或乱序事件、安全与活性条件——这是仓库里的工程原则，不是 Skein 的例外。
</div>

<div v-click class="tla-cloud">
  <span class="tla-tag">WAL Group Commit</span>
  <span class="tla-tag">Page Cache Admission</span>
  <span class="tla-tag">Transaction Concurrency</span>
  <span class="tla-tag">Concurrent Snapshots</span>
  <span class="tla-tag">Sparse Relational Activation</span>
  <span class="tla-tag">Sparse Relational Recovery</span>
  <span class="tla-tag">Index Publication</span>
  <span class="tla-tag">Index Statistics</span>
  <span class="tla-tag">Compaction Visibility</span>
  <span class="tla-tag">Generation Reclamation</span>
  <span class="tla-tag">Knowledge Retrieval Pipeline</span>
</div>

</div>

<!--
"64 这个数字不是重点，重点是覆盖面——WAL、页缓存、并发事务、压缩回收、复制，甚至上一页讲的 GraphRAG 查询路径，都有对应的模型。TLC 跑 Target 配置和至少一个 mutant/反例配置，两个都要过。"
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span class="active">03 How</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span>05</span></div>

# 第二支柱：查询引擎自己跟自己对账

<div class="deck-challenge-lede c2 text-sm leading-relaxed mt-3">
TLA+ 证明的是并发和崩溃恢复的状态机；优化器选错计划、执行器算错聚合，TLA+ 管不着。这一层交给 <strong class="c1">9 个差分 / 变形测试 oracle</strong>，思路来自 SQLancer 的 TLP（Ternary Logic Partitioning）。
</div>

<div class="stat-row">
  <div class="stat"><div class="stat__num">9</div><div class="stat__label">metamorphic oracle<br/>graph 4 + SQL 4 + 存储 1</div></div>
</div>

<div class="db-capability-grid mt-5">
  <div class="db-capability db-capability--graph">
    <div class="db-capability__name">Graph 侧</div>
    <div class="db-capability__role">Cypher 查询</div>
    <div class="db-capability__strength">✓ Plan-differential：memo 搜索 vs 兜底直接规划<br/>✓ Graph TLP / TLP-Aggregate：<code>Q</code> 拆成 <code>p</code> / <code>NOT p</code> / <code>IS NULL</code> 三份必须并回 <code>Q</code></div>
    <div class="db-capability__limit">另有谓词重写等价性、identifier 双射 + 关系方向反转的变形测试</div>
  </div>
  <div class="db-capability db-capability--sqlite">
    <div class="db-capability__name">SQL 侧</div>
    <div class="db-capability__role">PostgreSQL SQL / SQL-PGQ</div>
    <div class="db-capability__strength">✓ 同一套 TLP / TLP-Aggregate，换成带主键、可空列、索引、JOIN 的关系 schema<br/>✓ Join-rewrite：3-4 表 INNER/LEFT 树，有序 vs 无序结果必须一致</div>
    <div class="db-capability__limit">谓词重写等价性同样覆盖 SQL 侧</div>
  </div>
  <div class="db-capability db-capability--search">
    <div class="db-capability__name">存储</div>
    <div class="db-capability__role">on-disk 状态机</div>
    <div class="db-capability__strength">✓ 直接篡改 checkpoint + WAL 尾部的原始字节<br/>✓ Strict Append oracle：对照一个纯 watermark 模型，重复 / 乱序 / 类型不符 / 重启重放都要逐位一致</div>
    <div class="db-capability__limit">panic 判失败，带类型的 error 判正常</div>
  </div>
</div>

<div v-click class="callout mt-4 text-sm">
<strong class="c1">诚实的缺口</strong>：NoREC（另一种 SQLancer 技术）故意还没做——当前查询子集写不出 <code>SUM(CASE WHEN p THEN 1 ELSE 0 END)</code>，除非引入一条只服务测试、不服务生产的执行路径。宁可先承认缺口，也不为了凑技术清单造一条假路径。
</div>

</div>

<!--
"TLA+ 管的是状态机对不对，这九个 oracle 管的是——两条不同的路径算出来的结果，是不是同一个答案。SQLancer 这个思路懂的人应该不少：拿一个谓词，拆成真、假、空三份，加起来必须等于原集合。图查询和 SQL 查询都用这一招；存储层用的是另一种——直接对照一个理想化的模型逐位比对。"
- "NoREC 那句话是这页的重点：不是忘了做，是宁可先承认做不到，也不为了凑一条技术清单，去写一条只有测试用、生产用不到的假路径。"

[Sources]
- skein/crates/fuzz/README.md
-->

---
layout: center
class: deck-part-hero
---

<div class="text-center deck-section-hero">

<div class="progress-bar mb-8 justify-center"><span>01 Why</span><span class="dot">·</span><span>02 What</span><span class="dot">·</span><span>03 How</span><span class="dot">·</span><span class="active">04 Status</span><span class="dot">·</span><span>05 So What</span></div>

<div class="c4 text-sm tracking-widest uppercase mb-4">Part 4</div>

# 现状

<div class="c3 mt-4 text-lg">
已在 RSSledge dogfood，Mem 的稳定切换仍在验证
</div>

</div>

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span class="active">04 Status</span><span class="dot">·</span><span>05</span></div>

# 成熟度——诚实地讲

<div class="deck-split">

<div v-click>
  <div class="c1 font-semibold mb-2">RSSledge 已经在用</div>
  <div class="c2 text-sm leading-relaxed">
    RSSledge 把 Skein 作为<strong class="c1">唯一应用数据库</strong>：没有 SQLite 兼容层、双写或运行时切库。<br/>
    <span class="c3">它目前是 development / nightly consumer：真实 dogfood，不是稳定 / GA 客户流量。</span>
</div>
</div>

<div v-click>
  <div class="c1 font-semibold mb-2">Mem 的稳定切换仍然 gated</div>
  <div class="c2 text-sm leading-relaxed">
    Parser / AST → 内存态存储 → 逻辑计划 → Cascades 优化器 → 持久化 WAL 存储 → Ladybug 兼容层 → 分析钩子。<br/>
    <span class="c3">P0 是 Mem 的大规模生产验收：代表性副本存储验收、10 万+ 文档规模的向量检索对齐、向量索引召回率验收。</span>
  </div>
</div>

</div>

</div>

<!--
"两句话总结这一页：RSSledge 已经用 Skein 做真实 dogfood；但 Mem 的稳定/GA 激活是另一条更严格的路径，仍然受生产规模验证门槛约束。"
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span class="active">04 Status</span><span class="dot">·</span><span>05</span></div>

# 不只是设计，也有测得的数字

<div class="deck-challenge-lede c2 text-sm leading-relaxed mt-3">
这些是本地基准测试的结果，标注了机器和日期——<strong class="c1">方向性证据，不是生产环境的正式指标</strong>，跟"证明了什么，声称了什么"那页是同一个诚实标准。
</div>

<div class="stat-row mt-5">
  <div class="stat"><div class="stat__num">28.37x</div><div class="stat__label">单调追加快路径（batch=32）<br/>34/34 批次命中，零回退</div></div>
  <div class="stat"><div class="stat__num">3.1x</div><div class="stat__label">图一跳展开（LIMIT 50）<br/>10 万度节点 P50：844µs → 272µs</div></div>
  <div class="stat"><div class="stat__num">63.2x</div><div class="stat__label">零拷贝取值<br/>借用 ValueRef vs 拥有 Value 物化</div></div>
  <div class="stat"><div class="stat__num">2.86x</div><div class="stat__label">流式建索引<br/>峰值内存降低，同时还更快</div></div>
</div>

<div v-click class="mt-5 c2 text-sm leading-relaxed">
最喜欢的一条不是速度，是<strong class="c1">形状</strong>：图上一跳查询访问 10 万条边和访问 32 条边，延迟几乎一样（272µs vs 14.6µs，远小于度数 3,000 倍的差距）——因为执行器游标拿到 50 条结果就停，展开报告证明只碰过 50 条关系，不是碰过全部之后再截断。
</div>

</div>

<!--
"这几个数字特意标了机器型号和日期——不是产品发布指标，是我们自己盯着看的方向性证据。挑四个最有意思的：单调追加那条是发现大部分写入其实是'追加到页尾'这个特殊形状，为它单开一条快路径，34 个符合条件的批次全部命中，零回退；零拷贝那条 63 倍看着夸张，但它衡量的是'要不要多拷贝一份内存'，不是端到端应用查询提速，别混着理解。"
- "最后一句是这页的重点：一跳查询碰 10 万条边和碰 32 条边，几乎一样快。不是因为压缩了数据，是因为查询知道自己只要 50 条，游标碰到 50 条就退出——正好呼应前面 GraphRAG 那页'跳数上界'不是一句空话。"

[Sources]
- skein/docs/ROW_PAGE_MONOTONIC_APPEND_BENCHMARK.md（测于 2026-08-20，macOS arm64）
- skein/docs/EXECUTOR_MORSEL_BENCHMARK.md（测于 2026-08-17 / 2026-08-18，Apple M5 Max）
- skein/docs/SEARCH_GENERATION_BENCHMARK.md
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span class="active">04 Status</span><span class="dot">·</span><span>05</span></div>

# Nightly 灰度：先隔离发布，再进入运行模式

<div class="release-boundary mt-4">
  <div class="release-lane release-lane--stable">
    <div class="release-lane__eyebrow">Stable / 线上</div>
    <div class="release-lane__title">Skein 不编译进包</div>
    <div class="release-lane__body">没有实验引擎、没有设置入口，也不会改变现有 Kuzu / LanceDB / SQLite 数据路径。</div>
  </div>
  <div class="release-lane release-lane--nightly">
    <div class="release-lane__eyebrow">Nightly · 灰度中</div>
    <div class="release-lane__title">专用 Skein Nightly 产物，默认不启动</div>
    <div class="release-lane__body">当前由受审计的 bootstrap manifest 选择实验模式；Settings → Labs 是正在产品化的后续入口。</div>
  </div>
</div>

<div v-click class="rollout-steps rollout-steps--four">
  <div class="rollout-step">
    <div class="rollout-step__title">1 · Shadow read</div>
    <div class="rollout-step__desc">旧存储仍权威；Skein 只读并对比结果</div>
  </div>
  <div class="rollout-step">
    <div class="rollout-step__title">2 · Dual-write</div>
    <div class="rollout-step__desc">两边同时写；旧存储仍是读路径</div>
  </div>
  <div class="rollout-step">
    <div class="rollout-step__title">3 · Dual-read</div>
    <div class="rollout-step__desc">两边读并验证；旧存储继续主读</div>
  </div>
  <div class="rollout-step rollout-step--active">
    <div class="rollout-step__title">4 · Skein-only</div>
    <div class="rollout-step__desc">全量导入、验证和显式授权后，才成为唯一权威</div>
  </div>
</div>

<div v-click class="mt-5 callout c2 text-sm leading-relaxed">
<strong class="c1">当前灰度控制面是 bootstrap manifest；Labs 将把它产品化。</strong>模式选择在启动前写入受审计配置；运行时重新校验数据、投影与权限门槛。<br/>
<span class="c3">第一个迁移从 SQLite Content Store 开始——5 张表（<code class="text-xs">content_documents</code>、<code class="text-xs">thread_messages</code>、<code class="text-xs">content_chunks</code>、<code class="text-xs">content_anchors</code>、<code class="text-xs">content_migration_state</code>），落地为 Skein 内置的 PostgreSQL 方言关系存储——完全嵌入，不依赖任何 PostgreSQL 服务端或客户端库；图与向量在各自验证完成前不越过权威边界。</span>
</div>

</div>

<!--
"先看上面：Stable 与 Nightly rehearsal 的边界是构建边界，而不是一个运行时 if。线上包完全不带实验引擎；专用 Nightly 产物默认关闭，由 bootstrap manifest 显式启动。"
- "Labs 不是一个让用户随手换数据库的普通设置；它是正在产品化的实验控制面。当前先由 manifest 驱动：Shadow、Dual-write、Dual-read；Skein-only 仍受全量导入、结果验证和明确授权约束。"
- "四个名字对应运行时真实的权威边界：前三种模式都以旧存储为权威，只有 Skein-only 才会切换权威。"
- "SQLite Content Store 是第一个迁移目标，不是随便选的——5 张表覆盖了 Mem App 内容存储 schema v4 的全部范围，替代方案是 Skein 自带的 PostgreSQL 方言关系层，进程内嵌入，不需要额外起一个 PostgreSQL 实例。"

[Sources]
- Mem server, `nmem-rs/crates/nmem-server/Cargo.toml` and `src/skein_runtime.rs` (feature and runtime-mode boundaries)
- skein/docs/specs/POSTGRES_RELATIONAL_CONTENT_STORE_SPEC.md
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span class="active">04 Status</span><span class="dot">·</span><span>05</span></div>

# "受控灰度"不是一个开关，是 11 个独立就绪域

<div class="deck-challenge-lede c2 text-sm leading-relaxed mt-3">
上一页的四个阶段背后，是一份 schema 校验过的 JSON 就绪计划——不是一个人手动跑的脚本。每个域独立回答"我这部分能不能上"，而不是一个全局布尔值。
</div>

<div class="tla-cloud mt-4">
  <span class="tla-tag">graph</span>
  <span class="tla-tag">query</span>
  <span class="tla-tag">query_family</span>
  <span class="tla-tag">graph_route</span>
  <span class="tla-tag">search_route_ownership</span>
  <span class="tla-tag">storage</span>
  <span class="tla-tag">search_projection</span>
  <span class="tla-tag">search_projection_shadow</span>
  <span class="tla-tag">search_candidate_shadow</span>
  <span class="tla-tag">workload_fixture</span>
  <span class="tla-tag">background</span>
</div>

<div v-click class="mt-5 grid grid-cols-2 gap-x-8 gap-y-3 text-sm leading-relaxed">
  <div>
    <div class="c1 font-semibold mb-1">退出码是三态，不是布尔</div>
    <div class="c2"><code>0</code> 就绪 · <code>1</code> 完成但被阻塞 · <code>2</code> 执行失败——"被阻塞"是一个明确、可区分的结果，不是笼统的"没通过"。</div>
  </div>
  <div>
    <div class="c1 font-semibold mb-1">两档内存上限，不是一台跑分机的配置</div>
    <div class="c2"><code>desktop_bound_8_gib</code>（动态调度，实际常在 1-2 GiB）与 <code>capability_512_mib</code>（硬上限）——两档都要过。</div>
  </div>
</div>

<div v-click class="callout mt-4 text-sm">
留存的证据只存<strong class="c1">查询和参数的摘要</strong>，不存查询原文或结果行——就绪证明本身也要对隐私负责。
</div>

</div>

<!--
"上一页的 shadow / dual-write / dual-read / Skein-only，看起来像一个进度条。背后其实是 11 个互相独立的域，每一个都要单独回答'我这部分好了没'——图存储好了，不代表搜索投影好了，也不代表后台任务调度好了。"
- "退出码那条很喜欢：1 不是失败，是'做完了，但有已知阻塞'。这个区分本身就是诚实设计的一部分。"
- "两档内存上限也是故意的——8GB 那档是真实桌面机器的动态调度，512MB 那档是专门验证'低内存也能跑'的能力证明，不是同一件事的两种措辞。"

[Sources]
- skein/docs/PRODUCTION_GRAPH_STORAGE_QUALIFICATION.md and sibling PRODUCTION_*_QUALIFICATION.md docs
- skein/crates/readiness/src/lib.rs
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span class="active">04 Status</span><span class="dot">·</span><span>05</span></div>

# 两个真实的教训

<div class="incident-card">
  <div class="incident-card__title">① 把一样东西排除在构建之外，可能比包含进来还难</div>
  <div class="incident-card__body">
    一个跟 Skein 完全无关的 Bazel target，在全新 checkout 里连"分析"都通不过——因为 <code>MODULE.bazel</code> 无条件注册了私有的 <code>skein_src</code> 本地仓库，Bazel 在依赖裁剪发生之前就要先解析它。
  </div>
</div>

<div class="incident-card">
  <div class="incident-card__title">② 同一个事实，写在两个地方，一个会忘记更新</div>
  <div class="incident-card__body">
    一次发布被拦下：Skein 源码 pin 只在一个 workflow 里更新了，另一个 workflow 里重复的字面量 pin 没跟着更新。发布正确地被拦下，没有坏产物流出去。修复后 preflight 会交叉核对每个 workflow 的 pin 和根 gitlink——几天后下一次 pin bump，在分配 runner 之前就被自动拦住了。
  </div>
</div>

</div>

<!--
"这两个故事想传达同一件事：把一个新的存储引擎，以 git submodule 的方式先'藏在'仓库里但先不让它进生产构建，这件事本身的工程复杂度，不比让它进生产简单。"
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span class="active">04 Status</span><span class="dot">·</span><span>05</span></div>

# 接下来两件事

<div class="callout mt-3 c2 text-sm">
诚实标注：这两条现在都<strong class="c1">不在</strong> <code>skein/TODO.md</code> 里——是方向判断，不是已经立项的计划。放在这儿，是想现场听反馈。
</div>

<div class="deck-split mt-4">

<div v-click>
  <div class="flex gap-3">
    <div class="num">1</div>
    <div>
      <div class="c1 font-semibold">Agent trace 也是记忆的一部分</div>
      <div class="c2 text-sm mt-1 leading-relaxed">
        今天 Skein 统一的是图、向量、关系型内容。<br/>
        Agent 的执行轨迹——工具调用、会话历史——本质上也是同一种知识，不该活在一条单独的日志管道里，而应该被同一个引擎管理、检索、演化。
      </div>
    </div>
  </div>
</div>

<div v-click>
  <div class="flex gap-3">
    <div class="num">2</div>
    <div>
      <div class="c1 font-semibold">Agent 直接查库，靠 branch 隔离</div>
      <div class="c2 text-sm mt-1 leading-relaxed">
        今天 Agent 只能走 GraphRAG 安全查询路径：LLM 起草只读 Cypher，Skein 校验后执行。<br/>
        想让 Agent 能更直接地读写，同时给每个 Agent 一份像 git branch 一样<strong class="c1">可丢弃、可合并</strong>的隔离副本——探索性读写不污染主线。<br/>
        <span class="c3">现有的 MVCC 快照隔离（<code>SkeinConcurrentSnapshots</code>）解决的是事务读一致性，不是"可写、可丢弃的分支"——这是两回事，也是这条要补的差距。</span>
      </div>
    </div>
  </div>
</div>

</div>

</div>

<!--
"这一页跟前面不一样——前面讲的都是已经存在的东西，无论是已验证还是设计阶段。这两条是我自己的方向判断，仓库里的 TODO 里还没有。故意讲清楚这一点，是不想让大家觉得我在偷偷把愿景包装成路线图。"
-->

---
layout: center
class: deck-part-hero
---

<div class="text-center deck-section-hero">

<div class="progress-bar mb-8 justify-center"><span>01 Why</span><span class="dot">·</span><span>02 What</span><span class="dot">·</span><span>03 How</span><span class="dot">·</span><span>04 Status</span><span class="dot">·</span><span class="active">05 So What</span></div>

<div class="c4 text-sm tracking-widest uppercase mb-4">Part 5</div>

# 为什么这跟 Python 开发者有关系

<div class="c3 mt-4 text-lg">
诚实地说清楚：现在没有，将来可能有
</div>

</div>

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span class="active">05 So What</span></div>

# 先说清楚：Skein 现在没有 Python binding

<div class="mt-6 deck-pill-row">
  <span class="pill pill-muted">没有 pyo3</span>
  <span class="pill pill-muted">没有 maturin</span>
  <span class="pill pill-muted">没有 .pyi</span>
</div>

<div v-click class="mt-6 c2 text-base leading-loose">
这不是一个"能 <code>import skein</code>"的库——至少现在不是。<br/>
<span class="c3">如果今天的重点是"给 Python 用的 API"，这场分享到这里其实就该结束了。</span>
</div>

</div>

<!--
"我想先把这句话说完，再讲为什么还值得你们花时间听下去。"
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span class="active">05 So What</span></div>

# 但这是一个"Python 系统被 Rust 重写"的真实故事

<div class="deck-split">

<div v-click>
  <div class="c1 font-semibold mb-2">上一代后端，就是 Python</div>
  <div class="c2 text-sm leading-relaxed">
    Nowledge Mem 早期的后端是 Python（<code>nowledge-graph-py</code>），今天它是迁移参考，不再是产品运行时。
  </div>
</div>

<div v-click>
  <div class="c1 font-semibold mb-2">现在的主干是 Rust</div>
  <div class="c2 text-sm leading-relaxed">
    存储层、AI Agent 记忆层，从 Python 迁到 Rust——为了嵌入性、正确性、性能。<br/>
    Skein 是这条迁移路上，当前最深的一层。
  </div>
</div>

</div>

</div>

<!--
"很多在座的可能也在用 Python 搭 AI Agent 系统的存储/检索层。我们走过的这条路——从 Python 原型到 Rust 存储引擎——大概率不只是我们一家会走。"
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span class="active">05 So What</span></div>

# 嵌入式数据库这个模式，Python 生态最熟

<div class="deck-split">

<div v-click>
  <div class="c1 font-semibold mb-2">SQLite · DuckDB · LanceDB</div>
  <div class="c2 text-sm leading-relaxed">
    都是"链接进程"的库，不是要另外运维的服务。<br/>
    Python 开发者每天在用这个模式构建 AI 工具：本地向量库、本地 Agent 状态、本地缓存。
  </div>
</div>

<div v-click>
  <div class="c1 font-semibold mb-2">Skein 想做同一件事，但目标不同</div>
  <div class="c2 text-sm leading-relaxed">
    不是通用关系表，也不是纯向量表，而是<strong class="c1">图状的、给 AI Agent 用的记忆</strong>。<br/>
    面向的是构建 Python AI 工具链的<strong class="c1">系统 / 基础设施工程师</strong>，不是"直接 <code>import skein</code>"的应用开发者——至少现在不是。
  </div>
</div>

</div>

</div>

<!--
"这句话想留给大家：你们已经很熟悉'嵌入式数据库'这个形态了，只是习惯了它是关系表或者向量表。我们在赌的是——图状记忆，也值得有一个自己的 SQLite。"
-->

---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span class="active">Recap</span></div>

# Recap

<div class="mt-8 grid grid-cols-3 gap-12">

<div>
  <div class="c4 text-sm font-mono mb-3">01</div>
  <div class="c1 font-semibold text-base">为什么</div>
  <div class="c3 text-sm mt-2 leading-relaxed">三个数据库，一份记忆，协调的代价谁来付</div>
</div>

<div>
  <div class="c4 text-sm font-mono mb-3">02</div>
  <div class="c1 font-semibold text-base">是什么</div>
  <div class="c3 text-sm mt-2 leading-relaxed">嵌入式引擎，一套计划 / 优化器，两种查询语言，一条给 LLM 的安全查询路径</div>
</div>

<div>
  <div class="c4 text-sm font-mono mb-3">03</div>
  <div class="c1 font-semibold text-base">怎么保证正确</div>
  <div class="c3 text-sm mt-2 leading-relaxed">先建模再编码，诚实标注验证边界，两个真实的工程教训</div>
</div>

</div>

</div>

<!--
[15 秒快速过] 手势扫三列，不逐字念。"接下来是三句我自己的判断。"
-->

---
layout: two-cols
---

<div class="deck-slide-body">

<div class="progress-bar mb-2"><span>01</span><span class="dot">·</span><span>02</span><span class="dot">·</span><span>03</span><span class="dot">·</span><span>04</span><span class="dot">·</span><span class="active">Thoughts</span></div>

# Thoughts

<div class="deck-thesis-stack">

<div v-click class="deck-thesis-row">
  <div class="num">A</div>
  <div>
    <div class="c1 font-semibold">统一之后才看得见收益</div>
    <div class="c2 text-sm mt-1 leading-relaxed">分别优化三个系统，永远追不上一个共享 WAL 带来的正确性收益——这个收益在拆开的架构里根本无法被观测到。</div>
  </div>
</div>

<div v-click class="deck-thesis-row">
  <div class="num">B</div>
  <div>
    <div class="c1 font-semibold">形式化方法是护栏，不是营销</div>
    <div class="c2 text-sm mt-1 leading-relaxed">64 个 TLA+ 规格、9 个 fuzz oracle 听起来很唬人，但真正有价值的是那句"NoREC 我们故意还没做"——愿意公开讲边界，比讲覆盖率数字更重要。</div>
  </div>
</div>

<div v-click class="deck-thesis-row">
  <div class="num">C</div>
  <div>
    <div class="c1 font-semibold">嵌入式，是给 AI Agent 时代的答案</div>
    <div class="c2 text-sm mt-1 leading-relaxed">离线、进程内、按需付费的架构，可能比"再运维一个数据库服务"更贴近这个时代的负载形态。</div>
  </div>
</div>

</div>

</div>

::right::

<div class="deck-recap-links">

<header class="deck-recap-links__head">
  <div class="deck-recap-links__toprow flex flex-wrap items-center gap-x-3 gap-y-2 w-full min-w-0" aria-label="Nowledge Labs · Nowledge Mem">
    <div class="deck-recap-links__brandmarks flex flex-wrap items-center gap-x-2">
      <img src="./images/nowledge-labs-icon.png" alt="Nowledge Labs" class="cover-foot__logo cover-foot__logo--labs deck-recap-links__mark" />
      <span class="cover-foot__rule deck-recap-links__rule" aria-hidden="true"></span>
      <img src="./images/nowledge-mem-logo.webp" alt="Nowledge Mem" class="cover-foot__logo cover-foot__logo--mem deck-recap-links__mark" />
    </div>
  </div>
  <p class="deck-recap-links__kicker c4 text-[0.72rem] mt-1.5 tracking-[0.14em] uppercase">Nowledge Labs · Nowledge Mem</p>
</header>

<nav class="deck-recap-links__nav deck-recap-links__nav--org" aria-label="Nowledge links">
  <a class="deck-recap-links__item" href="https://www.nowledge-labs.ai/blog" target="_blank" rel="noopener noreferrer">
    <span class="deck-recap-links__icon c4" aria-hidden="true"><ph-article class="deck-recap-links__ph" /></span>
    <span class="deck-recap-links__body min-w-0">
      <span class="deck-recap-links__label c1">Labs blog</span>
      <span class="deck-recap-links__url c3 text-xs truncate">nowledge-labs.ai/blog</span>
    </span>
  </a>
  <a class="deck-recap-links__item" href="https://mem.nowledge.co/" target="_blank" rel="noopener noreferrer">
    <span class="deck-recap-links__icon c4" aria-hidden="true"><ph-globe class="deck-recap-links__ph" /></span>
    <span class="deck-recap-links__body min-w-0">
      <span class="deck-recap-links__label c1">Nowledge Mem</span>
      <span class="deck-recap-links__url c3 text-xs truncate">mem.nowledge.co</span>
    </span>
  </a>
  <a class="deck-recap-links__item" href="https://github.com/nowledge-co/community/" target="_blank" rel="noopener noreferrer">
    <span class="deck-recap-links__icon c4" aria-hidden="true"><ph-github-logo class="deck-recap-links__ph" /></span>
    <span class="deck-recap-links__body min-w-0">
      <span class="deck-recap-links__label c1">Community</span>
      <span class="deck-recap-links__url c3 text-xs truncate">github.com/nowledge-co/community</span>
    </span>
  </a>
</nav>

<div class="deck-recap-links__sep" role="presentation"></div>

<p class="deck-recap-links__group-label c4 text-[0.65rem] uppercase tracking-[0.16em] mb-1.5">Speaker · Weizhen Wang</p>

<nav class="deck-recap-links__nav deck-recap-links__nav--person" aria-label="Speaker links">
  <a class="deck-recap-links__item" href="https://github.com/hawkingrei" target="_blank" rel="noopener noreferrer">
    <span class="deck-recap-links__icon c4" aria-hidden="true"><ph-github-logo class="deck-recap-links__ph" /></span>
    <span class="deck-recap-links__body min-w-0">
      <span class="deck-recap-links__label c1">GitHub</span>
      <span class="deck-recap-links__url c3 text-xs truncate">github.com/hawkingrei</span>
    </span>
  </a>
</nav>

</div>

<!--
"三句判断念完，停一下。下一页谢幕。"
-->

---
layout: two-cols
---

<div class="deck-closing deck-closing--fin deck-thankyou-left h-full min-h-0 flex flex-col justify-center gap-6 pr-8 md:pr-10">

# 谢谢

#### Weizhen Wang @ Nowledge Labs

<div class="deck-closing-quote">
Skein 还没到生产。但"把三个数据库合并成一个嵌入式引擎"这个问题本身，值得现在就诚实地讲一遍——包括证明了什么，包括还没证明什么。
</div>

</div>

::right::

<div class="deck-recap-links">

<header class="deck-recap-links__head">
  <div class="deck-recap-links__toprow flex flex-wrap items-center gap-x-3 gap-y-2 w-full min-w-0" aria-label="Nowledge Labs · Nowledge Mem">
    <div class="deck-recap-links__brandmarks flex flex-wrap items-center gap-x-2">
      <img src="./images/nowledge-labs-icon.png" alt="Nowledge Labs" class="cover-foot__logo cover-foot__logo--labs deck-recap-links__mark" />
      <span class="cover-foot__rule deck-recap-links__rule" aria-hidden="true"></span>
      <img src="./images/nowledge-mem-logo.webp" alt="Nowledge Mem" class="cover-foot__logo cover-foot__logo--mem deck-recap-links__mark" />
    </div>
  </div>
  <p class="deck-recap-links__kicker c4 text-[0.72rem] mt-1.5 tracking-[0.14em] uppercase">Nowledge Labs · Nowledge Mem</p>
</header>

<nav class="deck-recap-links__nav deck-recap-links__nav--org" aria-label="Nowledge links">
  <a class="deck-recap-links__item" href="https://www.nowledge-labs.ai/blog" target="_blank" rel="noopener noreferrer">
    <span class="deck-recap-links__icon c4" aria-hidden="true"><ph-article class="deck-recap-links__ph" /></span>
    <span class="deck-recap-links__body min-w-0">
      <span class="deck-recap-links__label c1">Labs blog</span>
      <span class="deck-recap-links__url c3 text-xs truncate">nowledge-labs.ai/blog</span>
    </span>
  </a>
  <a class="deck-recap-links__item" href="https://mem.nowledge.co/" target="_blank" rel="noopener noreferrer">
    <span class="deck-recap-links__icon c4" aria-hidden="true"><ph-globe class="deck-recap-links__ph" /></span>
    <span class="deck-recap-links__body min-w-0">
      <span class="deck-recap-links__label c1">Nowledge Mem</span>
      <span class="deck-recap-links__url c3 text-xs truncate">mem.nowledge.co</span>
    </span>
  </a>
  <a class="deck-recap-links__item" href="https://github.com/nowledge-co/community/" target="_blank" rel="noopener noreferrer">
    <span class="deck-recap-links__icon c4" aria-hidden="true"><ph-github-logo class="deck-recap-links__ph" /></span>
    <span class="deck-recap-links__body min-w-0">
      <span class="deck-recap-links__label c1">Community</span>
      <span class="deck-recap-links__url c3 text-xs truncate">github.com/nowledge-co/community</span>
    </span>
  </a>
</nav>

<div class="deck-recap-links__sep" role="presentation"></div>

<p class="deck-recap-links__group-label c4 text-[0.65rem] uppercase tracking-[0.16em] mb-1.5">Speaker · Weizhen Wang</p>

<nav class="deck-recap-links__nav deck-recap-links__nav--person" aria-label="Speaker links">
  <a class="deck-recap-links__item" href="https://github.com/hawkingrei" target="_blank" rel="noopener noreferrer">
    <span class="deck-recap-links__icon c4" aria-hidden="true"><ph-github-logo class="deck-recap-links__ph" /></span>
    <span class="deck-recap-links__body min-w-0">
      <span class="deck-recap-links__label c1">GitHub</span>
      <span class="deck-recap-links__url c3 text-xs truncate">github.com/hawkingrei</span>
    </span>
  </a>
</nav>

</div>

<!--
收尾：
"存储引擎这种东西，通常是悄悄换掉的，没人会为它鼓掌。但我们觉得，怎么把三个数据库合并成一个、怎么证明它是对的、中间踩了什么坑——这件事本身值得现在就拿出来分享，而不是等它进了 GA 才讲。谢谢大家。"
-->
