---
layout: section
---

# Related Works

<!--
PLAN:
R0 section 分隔 — 聽到：接下來看前人；之後：準備進文獻。
R1 三個方向 — 看：A / B / C 三個小示意（E 和主體誰凍結、誰訓練），逐一出現；為何：這一節和最後的 Other Paths 都掛在這三個方向上，先給地圖；之後：知道今天的實驗是 A，B、C 是另外兩條路。
— 方向 A：固定 Embedding，訓練主體 —
R2 方向 A 分隔（glyph A morph 自地圖）。
R3 CLWE 對照組 — 大 stack 標「對照組」；聽到：方向 A 沒有正面在做的論文，最接近的是 Artetxe 2020 的 baseline CLWE；之後：A 在文獻裡只是個被比下去的 baseline。
R4–R5 Press & Wolf — tie 是我們架構前提，解綁消融的出處。
R6–R7 Wendler — logit lens 三段；為何：A 成立的前提是主體中段語言無關。
R8–R9 Semantic Hub — hub = 主體、spoke = embedding，就是 A 的賭注。
R10–R11 Chang — 語言 ≈ 均值偏移；為何：結果段的 mean-centering 與 donor 差距解讀都靠它。
— 方向 B：固定主體，訓練 Embedding —
R14 方向 B 分隔。
R15 Artetxe 2020 的主方法 MonoTrans Fig.1（stack 角落）— B 的原型，也是那篇論文的主角。
R16 B 的後續：de Vries、Active Forgetting、WECHSEL / FOCUS 四疊逐一出現，各一行；之後：知道 Other Paths 會回來細講。
— 方向 C：生出 Embedding —
R17 方向 C 分隔。
R18 MIMICK、ZeTT、HyperOFA 三疊逐一出現，各一行。
R19 收束 — 主角是問題「換成現代的 tied LM embedding 呢？」，CLWE 的評語退到 kicker 當背景 → 接 Setup。
-->

<!--
接下來是 Related Works。我不按年份、也不按主題分，而是按我們這個計畫本來的三個方向來排，因為每個方向都有一串前人在做，後面的實驗和最後的 Other Paths 也都掛在這三個方向上。
-->

---
layout: default
clicks: 2
---

# 三個方向

<div class="grid grid-cols-3 gap-6 mt-12">
  <div class="flex flex-col items-start gap-4">
    <OtherRouteGlyph route="A" vt="rw-route-a" :scale="0.85" />
    <div class="rw-dir"><b>固定 Embedding</b><br>訓練主體<span>今天的實驗</span></div>
  </div>
  <div v-click="1" class="flex flex-col items-start gap-4">
    <OtherRouteGlyph route="B" vt="rw-route-b" :scale="0.85" />
    <div class="rw-dir"><b>固定主體</b><br>訓練 Embedding</div>
  </div>
  <div v-click="2" class="flex flex-col items-start gap-4">
    <OtherRouteGlyph route="C" vt="rw-route-c" :scale="0.85" />
    <div class="rw-dir"><b>生出 Embedding</b><br>主體不動、零目標語料</div>
  </div>
</div>

<style>
.rw-dir { font-size: 22px; line-height: 1.4; }
.rw-dir b { font-weight: 600; }
.rw-dir span { display: block; margin-top: 0.4rem; color: var(--google-blue); font-size: 16px; font-weight: 600; }
</style>

<!--
整個計畫其實是在問同一件事：表層的 embedding 和底層的主體能不能拆開來各自處理。拆開之後就有三種做法。
A，固定 embedding、訓練主體：拿一組已經跨語言對齊好的 embedding 凍住，只用其他語言訓練主體，推論時把沒看過的語言接回去。這是今天實驗的主角。
[click] B 反過來，固定主體、只訓練新語言的 embedding，需要一些目標語料。
[click] C 更進一步，主體不動，也不拿目標語料，而是訓練一個網路直接把新 token 的 embedding 生出來。
每個方向我們都會看它的前人做到哪裡，A 會講得最細，B 和 C 先給一個輪廓，最後 Other Paths 再回來。
-->

---
layout: statement
kicker: 方向 A
---

<OtherRouteGlyph route="A" vt="rw-route-a" :scale="0.9" />

# 固定 Embedding，*訓練主體*

<!--
先看 A。這個方向沒有直接在做的論文，最接近的是 Intro 看過的、Artetxe 2020 裡的 CLWE 對照組。其餘幾篇不是在做同一件事，而是 A 能不能成立的前提：tie 的架構、主體中段是不是語言無關，以及語言之間的幾何差異長什麼樣。
-->

---
layout: figure-side
figureWidth: 44%
---

# 一個對照組：CLWE

<div class="max-w-[30rem] text-[var(--google-muted)]">Artetxe 2020 裡被拿來和 MonoTrans 比的 baseline</div>

<div class="mt-6 space-y-3">
<div>VecMap 對齊的靜態 subword 向量，凍結</div>
<div>只用英文訓練 BERT，推論時換上外語</div>
<div class="text-[var(--google-blue-dark)]">→ 我們換成現代 LM 的 tied embedding、causal LM</div>
</div>

<template #figure>
<PaperStack :pages="['/papers/artetxe2020/p1.jpg','/papers/artetxe2020/p2.jpg','/papers/artetxe2020/p3.jpg']" title="Artetxe, Ruder, Yogatama" venue="ACL 2020 · 對照組" width="13rem" vt="paper-artetxe2020" />
</template>

<!--
方向 A 沒有一篇論文是正面在做的。最接近的是 Intro 看過的 CLWE，但要再強調一次：它不是 Artetxe 這篇論文的方法，而是他們拿來和自己的 MonoTrans 比較的對照組之一，一個標準的 cross-lingual word embedding mapping 做法。
做法是凍結 VecMap 對齊的靜態 subword 向量，只用英文訓練 BERT，推論時換上外語的向量；任務越難掉得越多，作者在討論這個對照組時說，固定空間之間的映射「可能」太受限。
所以方向 A 在文獻裡的位置，其實只是一個被比下去的 baseline，沒有人用現代的條件認真做過。我們做的是同一個實驗骨架，但換掉了三件事：embedding 換成現代 LM 學出來的、架構換成 tied 的 causal LM、評估換成 likelihood。接下來幾篇就是這三個替換背後的依據。
-->

---
layout: figure-side
figureWidth: 44%
---

# Using the Output Embedding to Improve Language Models

Weight tying 的原始論文

<template #figure>
<PaperStack :pages="['/papers/press2017/p1.jpg','/papers/press2017/p2.jpg','/papers/press2017/p3.jpg']" title="Press & Wolf" venue="EACL 2017" width="13rem" vt="paper-press2017" />
</template>

<!--
第一個前提是 tie。Press 和 Wolf 在 EACL 2017 的這篇就是 weight tying 的出處。
Intro 講的「tied：同一個 E 讀進來、寫出去」，就是從這裡來的。
-->

---
layout: default
clicks: 3
---

<PaperStack class="!absolute top-6 right-12" :pages="['/papers/press2017/p1.jpg','/papers/press2017/p2.jpg','/papers/press2017/p3.jpg']" width="4.5rem" vt="paper-press2017" />

# Weight tying

<div class="text-[26px] mt-8">

$W_{\text{out}} = E$

</div>

<div class="max-w-[38rem] mt-8 space-y-3">
<div v-click="1">輸出矩陣本身就是好的 word embedding</div>
<div v-click="2">多種 LM 的 perplexity 顯著下降；NMT 可縮到一半以下</div>
<div v-click="3" class="text-[var(--google-blue-dark)]">→ 我們的架構前提；解除綁定另外做消融</div>
</div>

<!--
做法很簡單：讓輸出層的矩陣直接等於 input embedding。
[click] 他們先證明，LM 最上層的輸出矩陣本身就是一組有效的 word embedding，所以綁在一起是合理的。
[click] 結果是多種語言模型的 perplexity 顯著下降，NMT 模型可以縮到原本一半以下而不掉品質。他們也觀察到 tied embedding 的更新動態比較接近 untied 模型的輸出端。
[click] 對我們來說，這是整個主實驗的架構前提：因為綁定，推論時 L_ban 的列同時是輸入座標和輸出座標。最後我們也會做解除綁定的消融，看 tie 到底是不是必要的。
-->

---
layout: figure-side
figureWidth: 44%
---

# Do Llamas Work in English?

On the latent language of multilingual Transformers

<template #figure>
<PaperStack :pages="['/papers/wendler2024/p1.jpg','/papers/wendler2024/p2.jpg','/papers/wendler2024/p3.jpg']" title="Wendler et al." venue="ACL 2024" width="13rem" vt="paper-wendler2024" />
</template>

<!--
Wendler、Veselovsky、Monea、West 在 ACL 2024 問了一個很直接的問題：Llama 是不是用英文在思考？
他們在 Llama-2 上用 logit lens，把非英文 prompt 每一層的 hidden state 直接乘上 unembedding，看它在詞表上的分布。
-->

---
layout: figure-side
figureWidth: 42%
clicks: 3
---

<PaperStack class="!absolute top-6 left-16" :pages="['/papers/wendler2024/p1.jpg','/papers/wendler2024/p2.jpg','/papers/wendler2024/p3.jpg']" width="4.5rem" vt="paper-wendler2024" />

<div class="mt-44 space-y-3">
<div v-click="1"><b>input space</b>　離所有輸出 token 都很遠</div>
<div v-click="2"><b>concept space</b>　解出語意正確的 token，<b>英文版本機率較高</b></div>
<div v-click="3"><b>output space</b>　最後才移到輸入語言</div>
</div>

<template #figure>
<img src="/papers/wendler2024/fig.png" class="h-[26rem] rounded" />
<p>Wendler et al. 2024, Fig. 1</p>
</template>

<!--
右邊是他們的 Figure 1。任務是把法文的 "fleur" 翻成中文，所以正確答案是「花」；縱軸是層數，由下往上。
[click] 前面大約一半的層，解出來的東西都是雜訊，離所有輸出 token 都很遠，他們叫 input space。
[click] 到中間，開始解得出語意正確的 token，但機率比較高的是英文的 flower，不是中文的花，這是 concept space。
[click] 一直到最後幾層，才轉到中文的「花」，這是 output space。
這三段剛好對應我們的分工：輸入介面、主體、輸出介面。如果主體中段本來就在概念空間運作，換一組對齊好的 embedding 就有機會接上。
-->

---
layout: figure-side
figureWidth: 44%
---

# The Semantic Hub Hypothesis

LMs share semantic representations across languages and modalities

<template #figure>
<PaperStack :pages="['/papers/semantichub2025/p1.png','/papers/semantichub2025/p2.png','/papers/semantichub2025/p3.png']" title="Z. Wu et al." venue="ICLR 2025" width="13rem" vt="paper-semantichub2025" />
</template>

<!--
Zhaofeng Wu 等人在 ICLR 2025 把 Wendler 的觀察推得更遠，提出 semantic hub 假說。
這個名字借自神經科學的 hub-and-spoke 模型：大腦有一個整合語意的 hub，各種感官是連過來的 spoke。
-->

---
layout: default
clicks: 2
---

<PaperStack class="!absolute top-6 right-12" :pages="['/papers/semantichub2025/p1.png','/papers/semantichub2025/p2.png','/papers/semantichub2025/p3.png']" width="4.5rem" vt="paper-semantichub2025" />

# Hub and spoke

<div class="flex items-center gap-8 mt-2">
<div class="w-[24rem] shrink-0"><RelHubSpoke :step="$clicks >= 2 ? 1 : 0" /></div>
<div class="space-y-4 text-[21px]">
<div v-click="1">語意相同的輸入，中間層表徵相近；<b>介入 hub 會影響其他型態的輸出</b></div>
<div v-click="2" class="text-[#7e57c2] font-500">我們的賭注：hub 不變，換掉一根 spoke（embedding）也能用</div>
</div>
</div>

<!--
他們的圖像是這樣：中間有一個 hub，各種語言、程式碼、算式、影音都是連進來的 spoke。
[click] 證據有三層：不同語言中語意相同的輸入，在中間層的表徵相近，而且能用主要預訓練語言、透過 logit lens 解讀；這個現象延伸到算式、程式碼和影音；而且在一種型態上介入 hub，會可預期地影響其他型態的輸出，表示 hub 是模型真的在用的。
[click] 我們的整個主實驗可以用這張圖講完：hub 是主體，spoke 是 embedding。我們賭的是 hub 和語言無關，所以把中文這根 spoke 先拔掉訓練，推論時再接回去，hub 還是能用。
-->

---
layout: figure-side
figureWidth: 44%
---

# The Geometry of Multilingual Language Model Representations

<template #figure>
<PaperStack :pages="['/papers/chang2022/p1.jpg','/papers/chang2022/p2.jpg','/papers/chang2022/p3.jpg']" title="Chang, Tu, Bergen" venue="EMNLP 2022" width="13rem" vt="paper-chang2022" />
</template>

<!--
Chang、Tu、Bergen 在 EMNLP 2022 分析 XLM-R 在 88 種語言上的表徵幾何。
他們發現表徵可以分解成兩種軸：語言敏感軸，各語言的均值不同；以及語言中立軸，編碼位置、詞性這類跟語言無關的資訊。
-->

---
layout: figure-side
figureWidth: 50%
clicks: 2
---

<PaperStack class="!absolute top-6 left-16" :pages="['/papers/chang2022/p1.jpg','/papers/chang2022/p2.jpg','/papers/chang2022/p3.jpg']" width="4.5rem" vt="paper-chang2022" />

<div class="mt-44 space-y-4">
<div v-click="1">mean-centering 後，各語言子空間<b>高度重合</b></div>
<div v-click="2">平移語言均值 + 投影：目標語言 token <b class="text-[#ea4335]">10% → 47%</b></div>
</div>

<template #figure>
<img src="/papers/chang2022/fig.png" class="w-full rounded" />
<p>Chang et al. 2022, Fig. 4</p>
</template>

<!--
[click] 第一個關鍵結果：把各語言的均值減掉之後，它們的子空間高度重合。也就是說，語言之間很大一部分差異就是一個平移。
[click] 更直接的是右邊這張圖：只要把 hidden state 從語言 A 的均值平移到語言 B 的均值，模型就會開始預測語言 B 的 token。藍色是沒做任何事，Lang B 的 token 只佔 10%；紅色是平移加子空間投影，升到 47%，大約 4.7 倍。
這就是我們推論期 mean-centering 的方法來源：推論時把被禁用語言那一整群 embedding 列平移到訓練語言的均值附近，結果段會看到效果。
-->

---
layout: statement
kicker: 方向 B
---

<OtherRouteGlyph route="B" vt="rw-route-b" :scale="0.9" />

# 固定主體，*訓練 Embedding*

<!--
B 是 A 的反方向：主體凍住，只為新語言學一組 embedding。這個方向的前人比較多，我們挑和後面規劃直接相關的幾篇。
-->

---
layout: default
---

<PaperStack class="!absolute top-6 right-12" :pages="['/papers/artetxe2020/p1.jpg','/papers/artetxe2020/p2.jpg','/papers/artetxe2020/p3.jpg']" width="4.5rem" vt="paper-artetxe2020" />

# Artetxe 2020 的主方法：MonoTrans

<div class="max-w-[38rem] text-[var(--google-muted)]">凍結主體，只為 L<sub>2</sub> 學新的 token embedding</div>

<img src="/papers/artetxe2020/fig.png" class="mt-10 w-full rounded" />

<div class="text-[13px] text-[var(--google-muted)] mt-2">Artetxe et al. 2020, Fig. 1</div>

<!--
Artetxe 這篇論文真正的主角在這裡：MonoTrans。它分四步，圖從左到右看。(a) 先在英文上正常預訓練一個 BERT。
(b) 接著凍結主體，也就是圖中灰色的部分，只用 L2 的單語語料、一樣做 MLM，學一組新的 token embedding，綠色那一層。
(c) 再用英文做下游任務的 fine-tune；(d) 最後把 L2 的 embedding 換上去，直接 zero-shot。
注意方向：它是「凍結主體、學 embedding」，正好是方向 B 的原型，跟我們今天的主實驗剛好相反。他們的結論是 MonoTrans 和多語聯合訓練有競爭力，也就是說，這篇論文其實是方向 B 的正面證據。
-->

---
layout: default
clicks: 3
---

# B 的後續

<div class="grid grid-cols-3 gap-8 mt-8 text-[18px] leading-snug">
<div class="flex flex-col items-start gap-3">
<PaperStack :pages="['/papers/devries2021/p1.jpg','/papers/devries2021/p2.jpg','/papers/devries2021/p3.jpg']" title="de Vries & Nissim" venue="Findings of ACL 2021" width="7rem" vt="paper-devries2021" />
<div>GPT-2 版的 MonoTrans：主體不動，只重學 lexical embedding</div>
</div>
<div v-click="1" class="flex flex-col items-start gap-3">
<div class="flex gap-2">
<PaperStack :pages="['/papers/chen2023/p1.png','/papers/chen2023/p2.png','/papers/chen2023/p3.png']" title="Chen et al." venue="NeurIPS 2023" width="5rem" vt="paper-chen2023" />
<PaperStack :pages="['/papers/aggarwal2025/p1.jpg','/papers/aggarwal2025/p2.jpg','/papers/aggarwal2025/p3.jpg']" title="Aggarwal et al." venue="arXiv 2024" width="5rem" vt="paper-aggarwal2025" />
</div>
<div>Active Forgetting：預訓練時定期重置 embedding，讓主體好換</div>
</div>
<div v-click="2" class="flex flex-col items-start gap-3">
<div class="flex gap-2">
<PaperStack :pages="['/papers/wechsel2022/p1.jpg','/papers/wechsel2022/p2.jpg','/papers/wechsel2022/p3.jpg']" title="WECHSEL" venue="NAACL 2022" width="5rem" vt="paper-wechsel2022" />
<PaperStack :pages="['/papers/focus2023/p1.jpg','/papers/focus2023/p2.jpg','/papers/focus2023/p3.jpg']" title="FOCUS" venue="EMNLP 2023" width="5rem" vt="paper-focus2023" />
</div>
<div>新 token 的初始值 = 相近舊 token 的組合</div>
</div>
</div>

<div v-click="3" class="mt-8 text-[var(--google-blue-dark)]">→ 我們的問題：要多少目標語料，B 才追得上 A 的零語料表現？</div>

<!--
MonoTrans 之後，B 這條線大致往三個方向發展。
第一，de Vries 和 Nissim 把 MonoTrans 搬到 decoder：GPT-2 的 Transformer 層不動，只重學新語言的 lexical embedding。
[click] 第二，Chen 等人的 Active Forgetting 在預訓練時就定期把 embedding 重置，逼主體不要依賴某一組特定的 embedding，之後換語言會更容易；Aggarwal 等人把它搬到 decoder LLM。
[click] 第三，WECHSEL 和 FOCUS 處理的是起點：新 token 的 embedding 不要隨機初始化，而是用語意相近的舊 token 組合出來。
[click] 這幾篇都需要目標語料。我們在 B 想量的是一條曲線：要餵多少被禁用語言的語料，B 才追得上 A 完全不看目標語料的表現。細節最後 Other Paths 再講。
-->

---
layout: statement
kicker: 方向 C
---

<OtherRouteGlyph route="C" vt="rw-route-c" :scale="0.9" />

# 生出 *Embedding*

<!--
C 的想法是：與其為每個新語言學一組 embedding，不如訓練一個網路，輸入某種外部資訊，直接把新 token 的 embedding 生出來。主體不動，也不需要目標語料。
-->

---
layout: default
clicks: 2
---

# C 的前人

<div class="grid grid-cols-3 gap-8 mt-8 text-[18px] leading-snug">
<div class="flex flex-col items-start gap-3">
<PaperStack :pages="['/papers/mimick2017/p1.jpg','/papers/mimick2017/p2.jpg','/papers/mimick2017/p3.jpg']" title="MIMICK · Pinter et al." venue="EMNLP 2017" width="7rem" vt="paper-mimick2017" />
<div>從字元序列<b>模仿</b>出詞向量</div>
</div>
<div v-click="1" class="flex flex-col items-start gap-3">
<PaperStack :pages="['/papers/zett2024/p1.png','/papers/zett2024/p2.png','/papers/zett2024/p3.png']" title="ZeTT · Minixhofer et al." venue="NeurIPS 2024" width="7rem" vt="paper-zett2024" />
<div>Hypernetwork 從 byte 序列生成新 tokenizer 的 embedding</div>
</div>
<div v-click="2" class="flex flex-col items-start gap-3">
<PaperStack :pages="['/papers/hyperofa2025/p1.jpg','/papers/hyperofa2025/p2.jpg','/papers/hyperofa2025/p3.jpg']" title="HyperOFA · Özeren et al." venue="arXiv 2025" width="7rem" vt="paper-hyperofa2025" />
<div>輸入換成<b>外部對齊向量</b>，跨字元系統也能生</div>
</div>
</div>

<!--
C 的源頭是 MIMICK，用一個小的字元級模型去模仿詞向量，解決沒看過的詞。
[click] ZeTT 把它擴大成 hypernetwork：給它一個新 tokenizer，從每個 token 的 byte 序列生出 embedding，主體完全不動，而且在 Base 上訓練的 hypernetwork 可以直接套到 Instruct。限制是它看的是拼寫，換到不同文字系統就難了。
[click] HyperOFA 把輸入換成外部的多語對齊向量，才有辦法處理中文這種和訓練語言不共用字元的情況。
我們在 C 想測的，是這兩種輸入在我們禁用的語言上各自能走多遠，一樣放到 Other Paths。
-->

---
layout: statement
kicker: 小結 · CLWE 當年被評為太受限
---

# 換成現代的 *tied LM embedding* 呢？

<!--
把三個方向收起來：B 和 C 都有不少成功的前例，但都需要目標語料，或需要額外訓練一個生成網路。
A 是最省的一條路，什麼都不用加，但文獻裡最接近的例子，只是 Artetxe 2020 裡的 CLWE 對照組，作者對它的評語是可能太受限。
不過 A 的幾個前提，主體中段語言無關、語言差異主要是均值偏移，近幾年都有越來越多證據。
所以問題就是：換成現代的、在 LM 裡學出來的 tied embedding，這個結論還成立嗎？接下來進 Setup，看我們怎麼把這個問題變成可以量的實驗。
-->
