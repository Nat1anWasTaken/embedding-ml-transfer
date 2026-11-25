---
layout: section
---

# Introduction

<!--
PLAN:
1 section「Introduction」— 聽到：從幾個技術觀察出發；看：藍色分隔頁；之後：知道接下來是動機，不是天真問題。
2 觀察一：中間層不分語言 — 聽到 logit lens 在中間層解出英文概念；看：模型示意圖，click 後中間層冒出 "flower"；為何：先看端點再看中間，讓 aha 落在 click；之後：「中間是共享的概念空間」。
3 觀察二：tied — 聽到 input/output 是同一個 E；看：同一張示意圖（intro-model morph），click 後兩端 E 連成同一個矩陣、公式出現；之後：E 是主體唯一的讀寫介面。
4 語言 = 一組列 — 聽到「對主體來說，語言只是 E 的某幾列」；看：E 的放大版，按語言分色帶；之後：把語言想成可插拔的列。
5 思想實驗 — 聽到：如果訓練時這組列缺席、推論時放回？看：同一個矩陣（intro-rows morph），zh 列變虛線、click 後放回；之後：好奇「會不會動」。
6a 有人試過 — 接續思想實驗：大標「做法跟我們一樣」；看：左邊論文頁面展開，右邊「我們的思想實驗 = Artetxe 2020 的對照組」三步逐一對上；之後：「我們的思想實驗在那篇只是個 baseline」。
6a' 什麼是 CLWE — 聽到各語言用各自的 subword 詞表訓練 skip-gram，再用 VecMap（種子詞典 + 自我學習的正交映射）對到英文；看：論文縮到右上角，兩團詞向量，右邊三張卡片逐一出現，最後中文那團轉過去疊到英文上；之後：記住「靜態、各自訓練、事後線性對齊」。
6b 在哪裡失敗 — 聽到三個任務由易到難的結果；看：論文縮到右上角，CLWE vs MonoTrans 長條逐任務長出，最後只看英文；之後：「主體沒壞，壞在換詞向量，且越難越糟」。
6c 他們對 CLWE 的評語 — 聽到 overly restrictive 是對 baseline 的評語、不是論文結論，以及它是在什麼條件下說的；看：論文放大回左邊、右邊引文；之後：帶著懷疑進入「條件已經不同」。
7 為什麼值得再試 — 聽到他們用 2018 年的靜態詞向量 + encoder；看：兩欄比較表，逐列出現；之後：「條件已經不同了」。
8 假設 — 聽到研究問題一句話；看：大字問句，click 後兩個子假設（理解／生成）；之後：清楚知道今天要驗證什麼。
-->

<!--
這一節我們不從「LLM 能不能翻譯」這種很大的問題開始，而是從幾個已知的技術觀察出發。
三個觀察疊起來，會自然推出今天的假設：一個凍結、預先對齊好的 tied embedding，能不能讓主體零樣本地處理一個從沒看過的語言。
-->

---
layout: default
kicker: 觀察一
clicks: 1
---

# 中間層 *不太分語言*

<div class="w-[44rem]" style="view-transition-name: intro-model">
<IntroTiedDiagram mode="latent" :step="$clicks" />
</div>

<!--
第一個觀察來自近兩年的 interpretability 工作。拿一個多語 LLM，輸入法文的 "fleur"，要它輸出中文的「花」。
[click] 如果用 logit lens 去解碼中間層，會發現中間層最容易被解出來的不是法文也不是中文，而是英文的 "flower"。
Wendler 他們把這叫 latent English，Wu 等人把它推廣成 semantic hub：不同語言、甚至程式碼和算式，在中間層會落到相近的表徵。
細節我們在 Related Works 會一篇一篇看，這裡先記住一句話：Transformer 的中段，看起來是一個大致和語言無關的概念空間。
-->

---
layout: default
kicker: 觀察二 · Tied embedding
clicks: 2
---

# *同一個* E，讀也寫

<div class="w-[44rem]" style="view-transition-name: intro-model">
<IntroTiedDiagram mode="tied" :step="$clicks" />
</div>

<div v-click="2" class="text-[24px]">

$h_0 = E_{x} \qquad p(y \mid h_L) = \operatorname{softmax}(E\, h_L)$

</div>

<!--
第二個觀察是架構上的。很多小模型，像 Qwen3-0.6B、Llama 3.2 的 1B，用的是 weight tying。
[click] 也就是 input embedding 和輸出的 LM head 是同一個矩陣 E，沒有另外一個 W_out。
[click] 寫成式子就是：第一層吃進來的是 E 的第 x 列，最後一層的 hidden state 也是跟同一個 E 做內積再取 softmax。
所以在 tied 模型裡，E 是主體唯一的讀寫介面：主體從 E 讀進 token，也只能透過 E 把機率寫回 token。
-->

---
layout: default
kicker: 從主體的角度看
---

# 語言 = *E 的一組列*

<IntroEmbeddingRows state="all" />

<!--
把這兩個觀察放在一起，換個角度想：站在 Transformer 主體的位置，它其實不知道什麼叫「中文」。
它看到的只是：某些 token 進來時取 E 的某幾列，輸出時機率放在 E 的某幾列。
所以對主體來說，一個語言就是 E 裡面的一組列，英文是一組、西文是一組，中文是另一組。
如果這些列在幾何上本來就和其他語言對齊得很好，那主體需不需要真的看過這個語言？這就是我們想問的。
-->

---
layout: default
kicker: 思想實驗
clicks: 1
---

# *拿掉*，再*放回*

<div class="relative mt-10"><IntroEmbeddingRows :state="$clicks >= 1 ? 'returned' : 'banned'" /></div>

<!--
所以思想實驗是這樣：E 用一組已經對齊好的多語 embedding，並且把它凍結。
訓練主體的時候，把某個語言，我們叫它 L_ban，從資料裡完全拿掉，所以主體從頭到尾沒看過這組列。
[click] 到推論的時候，再把這組列放回去，讓 L_ban 的 token 直接走 E 進來、也從 E 出去。
[互動] 大家可以先在心裡下個注：放回去之後，主體能不能用上這組列？是完全沒反應、還是多少看得懂？
-->

---
layout: default
clicks: 3
---

<div class="grid grid-cols-[17rem_1fr] items-center gap-12 h-full pt-2">

<PaperStack :pages="['/papers/artetxe2020/p1.jpg','/papers/artetxe2020/p2.jpg','/papers/artetxe2020/p3.jpg']" title="Artetxe, Ruder, Yogatama" venue="ACL 2020" width="13rem" vt="intro-artetxe" />

<div>
  <div class="google-kicker">2020 年，有人當對照組試過</div>
  <h1 class="!mb-6">做法跟我們<em>一樣</em></h1>
  <div class="grid grid-cols-[1fr_auto_1fr] items-center gap-x-3 gap-y-3">
    <div class="text-[15px] font-semibold text-[#7e57c2]">我們的思想實驗</div>
    <div />
    <div class="text-[15px] font-semibold text-[var(--google-muted)]">Artetxe 2020 的對照組</div>
    <div class="google-card compact !py-3 text-[18px]">對齊好的 E</div>
    <div v-click="1" class="text-[22px] text-[var(--google-muted)]">=</div>
    <div v-click="1" class="google-card compact !py-3 text-[18px]">對齊好的詞向量</div>
    <div class="google-card compact !py-3 text-[18px]">凍結，只訓練主體</div>
    <div v-click="2" class="text-[22px] text-[var(--google-muted)]">=</div>
    <div v-click="2" class="google-card compact !py-3 text-[18px]">凍結，只用英文訓練</div>
    <div class="google-card compact !py-3 text-[18px]">推論時放回 L<sub>ban</sub></div>
    <div v-click="3" class="text-[22px] text-[var(--google-muted)]">=</div>
    <div v-click="3" class="google-card compact !py-3 text-[18px]">推論時換上外語向量</div>
  </div>
</div>

</div>

<!--
這個思想實驗其實有人做過，只是不是當主角。Artetxe、Ruder、Yogatama 在 ACL 2020 的 "On the Cross-lingual Transferability of Monolingual Representations" 裡，有一組對照組，做法幾乎就是剛剛的思想實驗。
先講清楚這篇論文的定位：它的主方法是 MonoTrans，方向剛好相反，是凍結主體、只學新語言的 embedding，這個我們 Related Works 和最後的 Other Paths 會再講。我們要看的是他們拿來比較的其中一個對照組。
左邊是我們剛剛的思想實驗，右邊一格一格對上去。
[click] 第一步，我們用一組已經對齊好的 E；他們也是先拿一組跨語言對齊好的詞向量。
[click] 第二步，我們凍結 E、只訓練主體；他們把這組向量凍結，只用英文在上面預訓練 BERT，再用英文標註資料 fine-tune，embedding 一直凍著。
[click] 第三步，我們推論時把 L_ban 的列放回去；他們推論時把 embedding 換成外語那一份，直接 zero-shot。
三步都對得上。也就是說，我們今天想做的事，在這篇論文裡只是一個被比下去的 baseline。那他們那組「對齊好的詞向量」是怎麼來的？下一張。
-->

---
layout: default
clicks: 2
---

<PaperStack class="!absolute top-6 right-12" :pages="['/papers/artetxe2020/p1.jpg','/papers/artetxe2020/p2.jpg','/papers/artetxe2020/p3.jpg']" width="4.5rem" vt="intro-artetxe" />

<div class="google-kicker">CLWE · Cross-lingual word embedding mappings</div>

# 各自訓練，*事後對齊*

<div class="grid grid-cols-[30rem_1fr] items-center gap-10 mt-2">

<IntroClweMap :mapped="$clicks >= 2" />

<div class="flex flex-col gap-3 text-[17px] mt-12">
  <div class="google-card compact !py-3"><b>skip-gram</b><br><span class="text-[15px] text-[var(--google-muted)]">每個語言各自的 subword 詞表、各自訓練</span></div>
  <div v-click="1" class="google-card compact !py-3"><b>VecMap · 起點</b><br><span class="text-[15px] text-[var(--google-muted)]">拼法相同的 subword 當種子詞典 D</span></div>
  <div v-click="2" class="google-card compact !py-3"><b>VecMap · 自我學習</b><br><span class="text-[15px] text-[var(--google-muted)]">用 D 解正交 W ⇄ 用最近鄰重找 D</span></div>
</div>

</div>

<div class="absolute bottom-8 left-14 text-[15px] text-[var(--google-muted)]">VecMap：Artetxe, Labaka, Agirre (ACL 2018) · 正交模式，所有語言對到英文</div>

<!--
這組向量在論文裡叫 CLWE，全名是 cross-lingual word embedding mappings：先各自學好單語的詞向量，再事後把它們映射到同一個空間。這是 2020 年以前做跨語言遷移的標準路線。
第一步，每個語言各自訓練 skip-gram。注意他們這裡每個語言用的是自己的 subword 詞表，和 MonoTrans 一樣，每個語言 32k，不是 mBERT 那種共用詞表。所以兩個空間形狀可能很像，狗和貓靠得近、花和樹靠得近，但彼此的座標系是任意的，就像畫面上中文那團被轉了一個角度。
接下來的對齊用的是 VecMap，Artetxe 自己 2018 年的工具。它先把向量做長度正規化、每一維置中、再正規化一次，然後分兩件事：
[click] 起點：需要一個初始的詞典 D，告訴它哪些詞對哪些詞。這篇表現最好的 ident 版本，用的是兩個語言裡拼法完全相同的 subword 當種子，像數字、人名、標點；另一個 unsup 版本則完全不用詞典，靠每個詞在自己語言裡的相似度分布去猜配對。
[click] 自我學習：給定 D，用 SVD 解出最好的正交矩陣 W，也就是 orthogonal Procrustes，W 等於 U V 轉置，其中 U Σ V 轉置是 X 轉置 D Z；再用映射後的最近鄰重新找一份 D；兩步交替直到收斂。VecMap 為了跳出壞的局部解，詞典那一步還會隨機丟掉一部分相似度，類似退火。
他們用的是正交模式，而且是把每個語言都對到英文，英文那邊不動。
這個方法不是這篇提出的，是他們拿來當對照組的標準 baseline。要記住它兩個特徵：向量是靜態、各自訓練的；對齊是事後用一個正交的線性映射找出來的。這兩點等一下會變成我們跟他們最大的差別。
接下來看這個對照組的表現。
-->

---
layout: default
clicks: 4
---

<PaperStack class="!absolute top-6 right-12" :pages="['/papers/artetxe2020/p1.jpg','/papers/artetxe2020/p2.jpg','/papers/artetxe2020/p3.jpg']" width="4.5rem" vt="intro-artetxe" />

# 對照組 CLWE：*任務越難，掉得越多*

<div class="mt-6 w-[46rem]">
<IntroArtetxeBars />
</div>

<!--
那這個對照組表現如何？我們把它和論文自己的主方法 MonoTrans 擺在一起看，三個跨語言任務，我照難度排。要記得藍色才是作者的方法，紅色是對照組。
[click] 最簡單的 MLDoc，文件分四類，CLWE 82.3，甚至比 MonoTrans 的 81.2 還高，是這個任務的最佳之一。
[click] 換成 XNLI，要判斷兩句之間的推論關係，CLWE 掉到 60.1，MonoTrans 是 67.8。
[click] 到最難的 XQuAD 抽取式問答，差距拉到 39.5 對 61.1；中文單獨看更慘，CLWE 只有 19.9，mBERT 是 58.0。
[互動] 在看下一個數字前，可以想一下：這是主體本身就學壞了，還是換上外語之後才壞的？
[click] 只看英文的話兩邊幾乎一樣，XNLI 82.4 對 83.1，XQuAD 84.2 對 83.9。所以主體沒問題，問題出在「換上外語詞向量」這一步，而且任務越需要細緻理解，掉得越多。
這些都是論文裡 Table 1 到 3 的全語言平均，CLWE 取他們最好的 768d ident 版本，MonoTrans 取最基本的 token emb 版本。
-->

---
layout: default
---

<div class="grid grid-cols-[17rem_1fr] items-center gap-12 h-full pt-6">

<PaperStack :pages="['/papers/artetxe2020/p1.jpg','/papers/artetxe2020/p2.jpg','/papers/artetxe2020/p3.jpg']" title="Artetxe, Ruder, Yogatama" venue="ACL 2020" width="13rem" vt="intro-artetxe" />

<div>
  <div class="google-kicker">他們對 CLWE 這個對照組的評語</div>
  <h1 class="!text-[52px] !leading-tight !mb-4">“<em>overly restrictive</em>”</h1>
  <p class="text-[var(--google-muted)]">mapping between two fixed embedding spaces may be overly restrictive</p>
</div>

</div>

<!--
先強調：這句不是整篇論文的結論。論文的結論是 MonoTrans 可行；這句是他們在討論 CLWE 這個對照組為什麼輸的時候講的。
原文是：CLWE 只在最簡單、最小的 MLDoc 上有競爭力，在 XNLI 和 XQuAD 這種比較難的任務就表現不好；他們的解讀是，在兩個固定的 embedding 空間之間做映射，可能太受限了，overly restrictive。
注意這裡是 may be，他們沒有說這條路不可能，而是說在那個條件下：靜態 subword 向量、事後對齊、encoder 加下游任務，它不夠。
接下來這張就是：這些條件到今天，有哪些已經不一樣了。
-->

---
layout: default
---

# 為什麼值得再試一次

<div class="mt-8">

| | Artetxe 2020 的 CLWE 對照組 | 我們 |
| --- | --- | --- |
| <span v-click="1">向量</span> | <span v-click="1">300 / 768 維靜態 subword skip-gram</span> | <span v-click="1">1024 維、LM 預訓練學出來的 embedding</span> |
| <span v-click="2">對齊</span> | <span v-click="2">事後用 VecMap 映射到英文</span> | <span v-click="2">同一個模型裡一起學出來</span> |
| <span v-click="3">模型</span> | <span v-click="3">BERT encoder，下游任務 fine-tune</span> | <span v-click="3">Tied causal LM，直接量 likelihood</span> |
| <span v-click="4">推論期</span> | <span v-click="4">直接換上外語向量</span> | <span v-click="4">另外試語言均值平移（mean-centering）</span> |

</div>

<!--
那為什麼值得再試一次？因為 2020 年那個設定和現在差很多。
[click] 第一是向量本身：那個對照組用的是 300 或 768 維的靜態 skip-gram 向量，我們用的是 LM 預訓練學出來的 1024 維 embedding。
[click] 第二是對齊方式：他們是兩個獨立訓練的空間事後用 VecMap 映射過去，我們的 embedding 是在同一個多語模型裡一起學出來的，對齊是模型自己長出來的。
[click] 第三是模型：他們是 encoder 加下游任務 fine-tune，我們是 tied 的 causal LM，E 同時是輸入和輸出。
[click] 第四，我們在推論時還會試把語言的均值偏移拿掉，這個想法來自後面會講的 Chang 等人的幾何分析。
-->

---
layout: default
kicker: 今天的假設
clicks: 2
---

# 放回去，*能用*嗎？

<div class="text-left text-[26px] leading-relaxed mt-2">
<div v-click="1"><b class="text-[var(--google-blue-dark)]">理解</b>　放回 L<sub>ban</sub> 的列，就能觸發零樣本理解</div>
<div v-click="2"><b class="text-[var(--google-blue-dark)]">生成</b>　再移除語言子空間偏移（mean-centering），觸發零樣本生成</div>
</div>

<!--
所以今天的假設可以濃縮成一句：用凍結、預先對齊的 tied embedding，主體從沒看過 L_ban，推論時把 L_ban 的列放回去，它能不能用？
[click] 拆成兩半。理解端：輸入和輸出共享同一個高維多語幾何空間，所以就算主體沒看過 L_ban，放回去應該就能觸發某種程度的零樣本理解。
[click] 生成端比較難：因為語言之間有均值偏移，主體可能不會主動把機率放到 L_ban 那群列上，所以我們預期要先把這個語言子空間的偏移拿掉，才會觸發生成。
這兩句話，就是後面所有實驗要檢驗的東西。接下來先看前人做到哪裡。
-->
