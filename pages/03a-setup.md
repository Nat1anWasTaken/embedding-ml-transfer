---
layout: section
---

# Experiments

Setup：先把尺做好

<!--
PLAN（順序即敘事：先決定「禁什麼」，再決定「怎麼量」，最後才是「訓練什麼、拿什麼比」）:
S1 section「Experiments」— 從假設走進實驗；要回答「放回去能不能用」得先做幾個決定。
S2 statement「放回去，怎麼知道*能用*？」— 路線圖寫成問題：禁哪些語言 · 怎麼量 · 跟誰比 · 訓練什麼 · E 從哪來 · 尺準不準。
S3 模型（SetupTiedModel）— 主角長什麼樣：300M tied decoder、Qwen3 詞表、主體只學英西德俄、E 凍結。後面所有東西都長在這張圖上。
S4 三個 L_ban 條件（SetupConditionGrid）— 因為主體只學英西德俄，我們能挑「沒看過」的語言；三格各拆一個變因。
S5 Fake-French 怎麼造 — 上一頁說「平移 ID」，這頁具體示範。
S6 Evaluation: FLORES+（NEW，標題直接點名資料集）— 同一句話、六種語言平行；示意句明標「示意」；為什麼平行才公平。
S7 BPB — 拿這些句子量什麼：每個 byte 的平均驚訝度；為何不用 PPL。
S8 一個 BPB 不夠：兩端與 τ（NEW，tau-line 空數線）— 3.6 是好是壞？需要「沒看過」與「看過」兩個參照模型 → τ。下界叫 Floor、上界叫 Upper，後面幾頁就是在造這兩個模型。
S9 Δ — τ 只量單語；翻譯要另一個量：給不給英文原句（3-shot）；同樣扣掉 Floor 那端。
S10 三份訓練集 — 主角與下界吃 Multi-Train；對照 Mono-Train；上界吃 Oracle-Train（= Multi + 中文 + Fake-French）。
S12 零洩漏 — 「主體沒看過」要靠資料保證。
S13 過濾後的結果：獨立檢查器 lid.176 抽查 0.94%，對照純英文的誤判底線 0.585% — 第一道關卡，呈現為最終過濾結果而非坑。
S14 E 的一組列 — 回到模型：三個條件在 E 裡各佔哪些列；接著用它造 τ 的兩端。
S15 Upper（B0-Upper）— Oracle-Train 上全可訓的 300M → τ = 1。
S16 Floor（B0-Lower）— Multi-Train 上訓練 + L_ban 列設成平均 μ → τ = 0；為什麼不直接用 B0-Lower（被 softmax 壓低）。
S17 凍結的 E 從哪來 — 兩端好了，回到主角：兩條軌道。
S18 Oracle Embedding — 不用另外訓練，就是 S15 B0-Upper 的 E。
S19 Donor Embedding（Qwen3）。
S20 拔哪一個 — 為什麼是 Qwen3-0.6B-Base、不是 Llama-3.2-1B；結尾「凍結就能開跑了嗎？」。
S21 這裡有坑：凍結了也會被壓制 — 呼應 S16 的「被壓低」。畫面是整個模型（SetupSoftmaxFlow：直立的 E → 主體 → h → Eᵀ → softmax），click 依序：正確答案列被拉上＋梯度式、L_ban 列成負例閃爍、L_ban 機率塌掉＋主體變紅、Gao 2019。
S22 修正：凍結 E，softmax 也要一起遮。
S24 語言內排序 vs 語言選擇 — 遮罩後要把 loss 拆開，才知道 τ 在量哪件事。
S25 τ：從 Floor 走到 Upper 的幾成（公式 + SetupTauMap + tau-line）— τ 的完整定義：公式與分子分母的意義、0 與 1 的讀法；click 把真實 Floor / Upper 數字放上 S8 的那條線。
S26 尺夠準嗎 — 兩道還沒過的關卡。
S27 BLI 關卡。
S28 量尺關卡 → 進結果。
-->

<!--
好，進入實驗。這一段我會刻意講得比較細，大概十分鐘，因為後面每一個結論、每一個數字，都建立在這裡的定義上。
我們這兩天跑完的是方向 A，也就是「凍結 embedding、只訓主體」那條線，加上它需要的基線和關卡。
如果等一下結果的部分有哪個數字讓你覺得怪，通常答案就藏在這一段的某個設計裡，所以這裡的坑我會先挖出來給大家看。
-->

---
layout: statement
kicker: Setup
---

# 放回去，怎麼知道*能用*？

禁哪些語言 · 怎麼量 · 跟誰比 · 訓練什麼 · E 從哪來 · 尺準不準

<!--
Intro 最後的假設是：主體從沒看過 L_ban，推論時把 L_ban 的列放回去，它能不能用。
要把這句話變成實驗，得先做幾個決定，這一段就照這個順序走。
第一，禁哪些語言。第二，「能用」要怎麼量成一個數字。第三，這個數字要跟誰比，才知道是好是壞。
然後才是：我們實際訓練了哪些模型、凍結的 E 從哪裡來，最後確認這把量尺本身夠不夠準。
-->

---
layout: figure
kicker: Setup · 模型
clicks: 2
---

# 一個 300M 的 tied decoder

Llama 3.2 風格 · Qwen3 詞表（151,936）· 主體只學英、西、德、俄

::figure::

<div class="w-[48rem]">
<SetupTiedModel :step="$clicks" />
</div>

<!--
先看主角長什麼樣。模型是我們自己的 mix-and-match 訓練框架，Llama 3.2 風格：RoPE、SwiGLU、GQA、RMSNorm，寬度 d=1024、16 個 head、4 個 KV head。
主力是 24 層的版本，我們叫它 300M，實際含 embedding 大約 4.75 億參數；另外有一個 12 層的 150M 拿來做 proxy 和掃超參。
[click] 最重要的是 tie_word_embeddings 開著：輸入查表的 E，和輸出算 logit 的 Eᵀ，是同一個矩陣。
[click] 方向 A 的所有主線實驗，這個 E 都是凍結的，能動的只有中間的 Transformer 主體。
詞表用的是 Qwen3 的 tokenizer，151,936 個 token，為什麼選它等一下講 E 從哪來的時候會交代。主體訓練時只看英文、西文、德文、俄文四種語言，每個 run 是一個節點 8 張 H200、跑 20B tokens。
既然主體只看這四種語言，下一個問題就是：我們要禁掉、留到推論時才放回去的是哪些語言？
-->

---
layout: figure
kicker: Setup · 條件
clicks: 5
---

# 三個 L<sub>ban</sub> 條件，同時禁用

::figure::

<SetupConditionGrid :step="$clicks" />

<!--
主體只學英西德俄，所以「主體沒看過的語言」我們可以自己挑。我們同時禁三個條件，排成一個 2×2 來看：橫軸是 token 主體有沒有見過，縱軸是和訓練語言的距離。
[click] 真法文：它的子詞和英、西、德大量共用，所以 token 主體見過，語言也近。主體訓練時沒有法文，但它的 token 早就在英西德裡出現過了。
[click] Fake-French：同樣是法文，但我們把 token ID 平移成全新的 ID，所以主體沒見過這些列，語言卻一樣近。
[click] 真中文：CJK 專屬的 token，主體沒見過，語言也遠。右下角那格「見過 token 但語言很遠」，我們沒有對應的條件。
[click] 這樣讀：Fake-French 對真法文，語言一樣，只差 token 有沒有重疊，量的是詞彙重疊效果。
[click] Fake-French 對真中文，都是全新 token，只差語言距離，量的是語言距離效果。原本的設計是「法文對中日文」，一次改了兩個變因，所以才引入 Fake-French；日文也移出主計畫，因為 XNLI 沒有日文，而且和中文共用漢字會模糊 V_ban 的邊界。
三個同時禁、共用同一組基線和 run，隱含的假設是「禁一個語言不會改變另一個語言的遷移」，我們認為風險不大，但它是一個假設。
三個裡面只有 Fake-French 是我們自己造的，下一頁看它怎麼造。
-->

---
layout: default
clicks: 4
---

<div class="flex items-center gap-4 mb-6">
  <div class="setup-ff-chip" style="view-transition-name: cond-fake">Fake-French</div>
  <h1 class="!m-0">怎麼造出來</h1>
</div>

<div class="absolute top-8 right-12">
  <PaperStack :pages="['/papers/dufter2020/p1.jpg','/papers/dufter2020/p2.jpg','/papers/dufter2020/p3.jpg']" width="4.5rem" vt="paper-dufter2020" />
</div>

<SetupFakeFrench :step="$clicks" />

<div v-click="4" class="mt-5 ml-[8.5rem] text-[16px] text-[var(--google-muted)]">
|S| = 29,622（涵蓋法文 99.5% 的 token 出現次數 + 256 個 byte token）· 更罕見的 token 先拆回 byte 再平移 · 序列只變長 2.2%
</div>

<style>
.setup-ff-chip {
  padding: 0.35rem 1rem;
  border: 2px solid var(--google-green-dark);
  border-radius: 0.8rem;
  background: color-mix(in srgb, var(--google-green-dark) 9%, white);
  color: var(--google-green-dark);
  font-size: 30px;
  font-weight: 700;
}
</style>

<!--
Fake-French 的方法來自兩篇前人：K 等人平移 unicode 碼位造 Fake-English，Dufter 和 Schütze 改成平移 token index。Qwen3 是 byte-level BPE，平移碼位的話私用區字元會被切成一串 byte token，所以我們只能平移 token ID。
[click] 做法是：法文照常用 Qwen3 切詞。這裡的切法是示意。
[click] 然後集合 S 裡的每一個 token t，都對應到 151,936 加上它在 S 裡的編號。字串長得一樣，但 ID 全新，主體從沒看過這些列。
[click] 有幾類 token 刻意不平移，當共用錨點：文件分隔的 endoftext、阿拉伯數字、空白、ASCII 標點。因為真中文和英文共用的也剛好是這幾類，Fake-French 保留同樣的錨點，兩個條件才可比；Dufter 他們也發現共用 special token 會影響多語能力。
[click] S 取涵蓋法文 99.5% token 出現次數的那些 token，再加 256 個 byte token，共 29,622 個；更罕見的先拆回 byte 再平移，序列只變長 2.2%。
所以 Fake-French 的每一句，其實就是一句真法文換了編號。三個條件都定好了，下一個問題是：放回去之後，要拿什麼句子、量什麼數字，才知道主體「能用」？
-->

---
layout: default
kicker: Setup · 評估
clicks: 3
---

# Evaluation: *FLORES+*

<div class="absolute top-8 right-12">
  <PaperStack :pages="['/papers/nllb2022/p1.png']" width="4.5rem" vt="paper-nllb2022" />
</div>

<div class="text-[17px] text-[var(--google-muted)] -mt-2">FLORES+ devtest：1,012 句，每一句都有 en / es / de / ru / fr / zh 的人工平行翻譯</div>

<div class="flores">
  <div class="fl-row tr"><span class="k">en</span><span class="t">The museum opens at nine in the morning.</span></div>
  <div class="fl-row tr"><span class="k">es</span><span class="t">El museo abre a las nueve de la mañana.</span></div>
  <div class="fl-row tr"><span class="k">de</span><span class="t">Das Museum öffnet um neun Uhr morgens.</span></div>
  <div class="fl-row tr"><span class="k">ru</span><span class="t">Музей открывается в девять утра.</span></div>
  <div class="fl-row fr" :class="{ hit: $clicks >= 1 }"><span class="k">fr</span><span class="t">Le musée ouvre à neuf heures du matin.</span><span class="tag">真法文</span></div>
  <div class="fl-row fake" :class="{ hit: $clicks >= 1 }"><span class="k">fake</span><span class="t">Le musée ouvre à neuf heures du matin.</span><span class="tag">同一句，ID 平移</span></div>
  <div class="fl-row zh" :class="{ hit: $clicks >= 1 }"><span class="k">zh</span><span class="t">博物馆早上九点开门。</span><span class="tag">真中文</span></div>
  <div class="fl-note">示意句，不是 FLORES+ 原文</div>
</div>

<div class="grid grid-cols-2 gap-6 mt-4 text-[17px]">
  <div v-click="2"><b>內容完全相同</b><br><span class="text-[var(--google-muted)]">分數的差別只來自語言和模型，不來自題目</span></div>
  <div v-click="3"><b>英文那句就是現成的「原文」</b><br><span class="text-[var(--google-muted)]">可以拿來測翻譯</span></div>
</div>

<style>
.flores { position: relative; display: flex; flex-direction: column; gap: 0.35rem; margin-top: 1rem; width: 44rem; }
.fl-row { display: grid; grid-template-columns: 3.6rem 1fr auto; align-items: center; gap: 0.8rem; padding: 0.32rem 0.8rem; border-radius: 0.6rem; font-size: 19px; transition: background 400ms ease, opacity 400ms ease; }
.fl-row .k { font-family: var(--slidev-code-font-family); font-size: 15px; color: var(--google-muted); }
.fl-row .tag { font-size: 14px; font-weight: 600; opacity: 0; transition: opacity 400ms ease; }
.fl-row.tr { color: var(--google-muted); }
.fl-row.hit .tag { opacity: 1; }
.fl-row.fr.hit { background: #fef7e0; }
.fl-row.fr .tag { color: #b06000; }
.fl-row.fake.hit { background: color-mix(in srgb, var(--google-green-dark) 10%, white); }
.fl-row.fake .tag { color: var(--google-green-dark); }
.fl-row.zh.hit { background: #fce8e6; }
.fl-row.zh .tag { color: var(--google-red); }
.fl-note { position: absolute; right: -9rem; top: 0.3rem; font-size: 13px; color: var(--google-muted); writing-mode: horizontal-tb; }
</style>

<!--
「能用」要變成數字，第一步是挑一批句子。我們用的是 FLORES+，NLLB 團隊整理的多語評估集：同一批英文句子，請人翻成兩百多種語言。我們用 devtest 的 1,012 句，取其中英、西、德、俄、法、中六種。
畫面上這句是我自己寫的示意句，不是 FLORES+ 原文，但格式就是這樣：一句話，每種語言一行，彼此是逐句對應的翻譯。
[click] 我們真正計分的是這三行：真法文、Fake-French、真中文。Fake-French 那一行就是剛剛講的，把法文那句切詞之後平移 ID，字串完全一樣、編號全新。
[click] 為什麼一定要平行句？因為六種語言講的是同一件事，題目的難度完全一樣。兩個模型、或兩個語言條件比較時，分數的差別只可能來自語言和模型本身，不會是「剛好抽到比較難的句子」。
[click] 第二個好處是：每一句中文都有一句對應的英文。英文那句可以直接當作「原文」，拿來測模型會不會翻譯，等一下 Δ 就用這個。
另外也會看 FineWeb-2 的 test split 當補充，不過主要的數字都來自 FLORES+。
-->

---
layout: default
kicker: Setup · 評估
clicks: 2
---

# 量什麼：*BPB*，不比 PPL

<div class="absolute top-8 right-12">
  <PaperStack :pages="['/papers/nllb2022/p1.png']" width="4.5rem" vt="paper-nllb2022" />
</div>

單獨讀 L<sub>ban</sub> 一句（無上下文），逐 token 算 −log p，攤到每個 byte

<div v-click="1" class="text-[26px] mt-4">

$$
\text{BPB} = \frac{\sum \text{NLL}}{\ln 2 \cdot \sum \text{UTF-8 bytes}}
$$

</div>

<div v-click="2" class="text-[18px] text-[var(--google-muted)] mt-2">越低越好 · 中文一字 3 bytes、法文字母 1 byte → token PPL 無法跨語言比</div>

<!--
有了句子，要量什麼？我們讓模型讀 L_ban 的那一句，看它對每個 token 有多「意外」，也就是 negative log-likelihood。
[click] 寫成式子：一千多句的 NLL 全部加起來，除以 ln 2 乘上這些句子原始的 UTF-8 byte 總數，單位是 bits per byte，BPB。注意是「和的比」，不是逐句比值的平均。Fake-French 的分母用原始法文的 byte 數，所以它和真法文可以逐句直接相減。
[click] 為什麼不用 token perplexity？因為同一個 tokenizer 下，不同語言每個 byte 對應的 token 數差很多，中文一個字就是 3 個 byte。攤到 byte 上，才是一把跨語言共用的尺。BPB 越低，代表模型越會預測這個語言。
訓練語言那邊我們也會盯著：L_train 的 BPB 是 FLORES+ 的英西德俄四項，加上 FineWeb-2 test 的西德俄三項，七項平均，用來確認主體在訓練語言上沒有變差。
統計上，所有差異都用句子層級的 bootstrap 一萬次算 95% CI，事先規定區間不含 0 才算有差。300M 這個規模，決策只看 likelihood 類的指標；XNLI、chrF++ 這些都延到更大的模型。
-->

---
layout: figure
kicker: Setup · 評估
clicks: 2
---

# 一個 BPB 不夠：要有*兩端*

<div class="grid grid-cols-2 gap-6 w-[46rem] mt-2 text-[18px]">
  <div v-click="1"><b class="text-[var(--google-gray)]">Floor · τ = 0</b><br><span class="text-[var(--google-muted)]">主體沒看過 L<sub>ban</sub>，L<sub>ban</sub> 列也不帶任何資訊</span></div>
  <div v-click="1"><b class="text-[var(--google-blue-dark)]">Upper · τ = 1</b><br><span class="text-[var(--google-muted)]">同一個架構，訓練時就看過 L<sub>ban</sub></span></div>
</div>

<div v-click="2" class="text-[22px] mt-3">

$\tau = \dfrac{\text{BPB}_{\text{floor}} - \text{BPB}_x}{\text{BPB}_{\text{floor}} - \text{BPB}_{\text{upper}}}$

</div>

::figure::

<div class="mx-auto" style="width: 46rem; view-transition-name: tau-line">
<RevealChart
  :min="-0.2" :max="1" axis="τ"
  :rows="[
    { label: '真中文', value: 0, color: 'red', at: 99 },
    { label: 'Fake-French', value: 0, color: 'green', at: 99 },
  ]"
/>
</div>

<!--
可是一個 BPB 本身沒有意義。假設我們的模型中文 BPB 是 3.6，這是好還是壞？中文本來就比英文難預測，3.6 可能已經很好，也可能很糟。
[click] 所以要兩個參照點。一端叫 Floor：主體一樣沒看過 L_ban，而且 L_ban 的列完全不帶資訊，這是「什麼都沒遷移」的樣子。另一端叫 Upper：同一個架構，但訓練時就看過 L_ban，這是「完全學會」的樣子。
[click] 我們的模型落在兩端之間的哪裡，就是遷移比例 τ：Floor 的 BPB 減掉我們的 BPB，除以 Floor 減 Upper。τ = 0 是什麼都沒遷移，τ = 1 是跟看過 L_ban 的模型一樣好。
下面這條就是 τ 數線，現在兩端還是空的。接下來 setup 的大半，其實就是在把這兩個參照模型造出來：Upper 是哪個模型、Floor 要怎麼做，做的過程中還踩了幾個坑。
-->

---
layout: figure
kicker: Setup · 評估
clicks: 4
---

# Δ：給了原文，譯文*好猜*多少

FLORES+ dev 前 3 句當 3-shot 範例 → 接來源句 x → 只對參考譯文 y 計分

::figure::

<div class="w-[50rem]">
<SetupDelta :step="$clicks" />
</div>

<div v-click="4" class="text-[18px] mt-4 text-center">
和 τ 一樣扣掉 Floor 那端：回報 <b>Δ − Δ<sub>floor</sub></b>　<span class="text-[var(--google-muted)]">（Floor 也能抄數字、拉丁字母人名）</span>
</div>

<!--
τ 只量單語：模型會不會預測一句 L_ban。可是我們的假設講的是「讀懂」和「寫出來」，這要用平行句的第二個好處：每一句都有對應的英文原文。
做法是 teacher-forced 的翻譯 likelihood：前面放 FLORES dev 的前 3 句當範例，格式是「原文、換行、譯文、空一行」重複三次，接著放這一題的來源句 x，然後只對參考譯文 y 的 BPB 計分。
[click] 對照組把 x 換成 x′：長度相近、但內容不相干的另一句。x′ 取的是按 byte 長度排序後的鄰居，所以長度幾乎一樣。
[click] Δ 就是兩者 BPB 的差：如果主體真的讀懂了來源句，給對的 x 時 y 應該比較好預測，Δ 就大於 0。
[click] 方向分兩種。L_ban 到英文：輸出是英文，繞過 tied 輸出端，量的是主體有沒有讀懂 L_ban，也就是理解端。英文到 L_ban：輸出要走 L_ban 的列，量的是生成端。這正好對上 Intro 假設的兩半。
[click] 最後和 τ 一樣要扣掉 Floor 那端：就算是 Floor，也能從原文抄數字和拉丁字母的人名，這部分跟對齊無關。例如 Floor 的 fr→en Δ 已經有 0.202，zh→en 只有 0.029。
好，量法定了：τ 和 Δ。接下來就是訓練哪些模型，包括 Floor 和 Upper 這兩端要怎麼造出來。
-->

---
layout: figure
kicker: Setup · 資料
clicks: 3
---

# 三份訓練集

FineWeb（英文）＋ FineWeb-2（非英文），單位：Qwen3 tokens

::figure::

<div class="w-[48rem]">
<SetupDataBars :step="$clicks" />
</div>

<!--
要造的模型有三種：我們的主角、Floor 用的下界模型、Upper 的上界模型。它們吃的訓練資料如下。
資料來源是標準版 FineWeb 加上 FineWeb-2。刻意不用 FineWeb-Edu，是為了跟 FineWeb-2 一樣都是通用網頁分布，避免 domain shift。
主力訓練集叫 Multi-Train，20.2B tokens：英文 40%，西文、德文、俄文各 20%。主角和下界模型都吃這一份。非英文比例拉高，是為了之後能回答「多語主體有沒有幫助」；放俄文是讓主體看過非拉丁字母。
[click] 對照組 Mono-Train 是 20.2B 的純英文，結果段拿來問「多語主體有沒有幫助」。
[click] 第三份是上界模型用的 Oracle-Train：它的前 20.2B 就是 Multi-Train，同一批文件、逐 shard 相同。
[click] 然後再「加上」中文 2.02B 和 Fake-French 2.02B，總共 24.2B。用加的、不用替換的，是為了讓上界和下界看到的 L_train token 完全一樣，唯一差別就是多出來的 L_ban。上界模型我們叫 B0-Upper，下界模型叫 B0-Lower。
加上去的只有中文和 Fake-French，沒有真法文：真法文從頭到尾不在任何訓練集裡，連上界模型都沒看過，所以它只拿來評估，看的是 BPB 本身。
訓練資料定了，下一件事是：「主體沒看過 L_ban」這句話要怎麼保證。
-->

---
layout: default
clicks: 1
---

# 零洩漏：過濾後*只少了一點*

<div class="setup-filters">
  <span><b>文件級</b> GlotLID 逐行判語言</span>
  <span class="plus">+</span>
  <span><b>Token 級</b> 含中文 token 就整篇丟</span>
</div>

<div v-click="1" class="setup-removed">
  <div class="setup-removed-head">
    <span></span><span>刪掉的 token</span><span>文件</span>
  </div>
  <div v-for="r in [
      { lang: '英文', tok: 1.86, doc: 0.71 },
      { lang: '西文', tok: 4.77, doc: 1.08 },
      { lang: '德文', tok: 2.81, doc: 0.90 },
      { lang: '俄文', tok: 1.66, doc: 0.41 },
    ]" :key="r.lang" class="setup-removed-row">
    <span class="lang">{{ r.lang }}</span>
    <span class="bar"><i :style="{ width: (r.tok / 6 * 100) + '%' }" /><em>{{ r.tok.toFixed(2) }}%</em></span>
    <span class="doc">{{ r.doc.toFixed(2) }}%</span>
  </div>
</div>

<style>
.setup-filters {
  display: flex;
  align-items: center;
  gap: 0.9rem;
  margin-top: 1.6rem;
  color: var(--google-muted);
  font-size: 18px;
}
.setup-filters span:not(.plus) {
  padding: 0.35rem 0.9rem;
  border: 1px solid var(--google-border);
  border-radius: 999px;
  background: white;
}
.setup-filters b { margin-right: 0.35rem; color: var(--google-blue-dark); font-weight: 600; }
.setup-removed { margin-top: 2.6rem; width: 40rem; }
.setup-removed-head,
.setup-removed-row {
  display: grid;
  grid-template-columns: 4rem 1fr 5rem;
  align-items: center;
  gap: 1.2rem;
}
.setup-removed-head { color: var(--google-muted); font-size: 15px; margin-bottom: 0.6rem; }
.setup-removed-head span:last-child,
.setup-removed-row .doc { text-align: right; }
.setup-removed-row { height: 3rem; font-size: 20px; }
.setup-removed-row .lang { font-weight: 600; }
.setup-removed-row .bar { position: relative; display: flex; align-items: center; height: 1.2rem; background: var(--google-surface); border-radius: 999px; }
.setup-removed-row .bar i { display: block; height: 100%; background: var(--google-blue); border-radius: 999px; transition: width 700ms cubic-bezier(0.2, 0.8, 0.2, 1); }
.setup-removed-row .bar em { margin-left: 0.7rem; color: var(--google-ink); font-size: 17px; font-style: normal; font-weight: 600; }
.setup-removed-row .doc { color: var(--google-muted); font-size: 17px; }
</style>

<!--
整個實驗的前提是「主體從沒看過 L_ban」，這句話要成立，資料必須真的乾淨，所以 Multi 和 Mono 兩份訓練集都過了兩層濾網：文件級逐行判語言，再加上 token 級，只要含一個中文 token 就整篇丟掉。
[click] 過濾的代價很小：英文只刪掉 1.86% 的 token、0.71% 的文件；最多的是西文，也只有 4.77% 的 token。
濾完之後，訓練集裡中文 token 和 Fake-French 的 ID 一次都沒有出現，所以主體確實沒看過這些列。
但這是濾網自己說的。我們事先規定，開跑前還要過一道「資料關卡」：換一個獨立的檢查器再抽查。
-->

---
layout: figure
kicker: 第一道關卡 · 資料
clicks: 3
---

# 過濾後的結果：殘留 *0.94%*

獨立檢查器 fastText lid.176 抽查，每份 100k 篇

::figure::

<div class="w-[46rem]">
<RevealChart
  :min="0.4" :max="1.1" :digits="3" axis="被判為法文／中文的文件比例（%）"
  :markers="[{ value: 1.0, label: '門檻 1%', color: 'red' }]"
  :rows="[
    { label: 'Multi-Train', value: 0.941, ci: [0.883, 1.003], color: 'blue', at: 1 },
    { label: 'Mono-Train（純英文）', value: 0.585, ci: [0.540, 0.634], color: 'gray', at: 2 },
  ]"
/>
</div>

<div v-click="3" class="text-center text-[19px] mt-2">
大部分是檢查器自己的誤判（純英文也有 0.585%）→ <b>開跑</b>
</div>

<!--
這是過濾完之後的最終結果，也是開跑前的第一道關卡。濾網自己說乾淨還不夠，我們用一個獨立的檢查器 fastText lid.176 再抽查一次：每份訓練集抽十萬篇，逐段判語言，事先規定洩漏率的 95% CI 上界要低於 1%。
[click] Multi-Train 被判成法文或中文的比例是 0.941%，CI 是 0.883% 到 1.003%。
[click] 這個數字要配著對照看：Mono-Train 是純英文，理論上不可能有法文或中文，lid.176 還是報了 0.585%。這就是檢查器自己的誤判底線。再拆下去，20 字元以上的行兩邊幾乎一樣，都是 0.29%；差距全部在 20 字元以下的短行，推測是把西文、德文的短片段誤判成法文。
[click] 所以 0.94% 裡大部分是檢查器自己的誤判，真正殘留的量很少，我們判定可以開跑。誠實補一句：CI 上界 1.003% 比事先寫的門檻多了 0.003 個百分點，照字面是沒過，我們是看了這個對照之後才判 Go，紀錄裡有附註。
對下游的影響是：真法文條件可能殘留少量短片段，可能讓真法文的結果有很小的正偏誤。
資料講完，回到模型本身：三個條件在 E 裡各是哪些列。
-->

---
layout: figure
kicker: Setup · Embedding
clicks: 3
---

# 對主體來說，語言 = *E 的一組列*

詞表：Qwen3 151,936 ＋ Fake-French 平移段 → 補齊到 181,632

::figure::

<div class="w-[50rem]" style="view-transition-name: setup-ematrix">
<SetupEMatrix mode="lang" :step="$clicks" />
</div>

<!--
Intro 講過，在 tied 模型裡，一個語言對主體來說就是 E 的一組列。現在三個條件都定了，可以把真實的列數畫出來，橫軸是 token ID。
[click] 大部分是 L_train 共用的列，也就是英、西、德、俄會用到的 token。
[click] 真中文的 V_ban 是 28,677 列，它們不是連續一段，而是散落在整個 Qwen3 詞表裡。這張圖的位置是示意，比例是對的。
[click] Fake-French 是我們另外加的 29,622 列，ID 從 151,936 排到 181,557，接在原本詞表後面，再補齊到 128 的倍數 181,632。
接下來所有的操作，不管是 Floor、donor 還是遮罩，都可以想成是「對這幾組列做了什麼」。
先用這張圖把 τ 數線的兩端造出來：上界模型和下界模型。
-->

---
layout: figure
kicker: Setup · 上界
clicks: 1
---

# τ = 1 的那一端：Upper

B0-Upper：同一個 300M 架構，在 Oracle-Train 上訓練

::figure::

<div class="w-[38rem]">
<SetupDataBars only="oracle" />
</div>
<div class="text-[15px] text-[var(--google-muted)] -mt-1 mb-2">↓ embedding 隨機初始化、全部可訓</div>
<div class="w-[50rem]" style="view-transition-name: setup-ematrix">
<SetupEMatrix mode="lang" :step="3" />
</div>

<div v-click="1" class="text-[19px] mt-3 text-center">看過 L<sub>ban</sub> 的模型　→　τ 數線上的 <b>1</b></div>

<!--
先造 τ = 1 那一端。上界模型叫 B0-Upper：同樣 300M、同一個主體架構，吃的是剛剛的 Oracle-Train，也就是 Multi-Train 再加上中文和 Fake-French。
它的 embedding 隨機初始化、全部可訓，跟一般預訓練一樣，所以這張矩陣的每一列，包括中文和 Fake-French 的列，都真的被訓練過。
[click] 它看過 L_ban，代表「完全學會」的樣子：它在 FLORES+ 上的 BPB，就是 τ 公式裡的 BPB_upper，也就是 τ = 1。
先記住這個模型，它等一下還會以另一個身分回來。
-->

---
layout: figure
kicker: Setup · 下界
clicks: 2
---

# τ = 0 的那一端：Floor

B0-Lower（沒看過 L<sub>ban</sub> 的 300M）訓練完，把 L<sub>ban</sub> 列設成平均值 μ

::figure::

<div class="w-[50rem]" style="view-transition-name: setup-ematrix">
<SetupEMatrix mode="floor" />
</div>

<div v-click="1" class="text-[19px] mt-4 text-center">
為什麼不直接用 B0-Lower？
</div>
<div v-click="2" class="text-[17px] mt-1 text-center text-[var(--google-muted)]">
L<sub>ban</sub> 列從來不是答案 → 被 softmax 一路壓低、被 weight decay 縮小 → BPB 差得不自然 → 灌水所有 τ
</div>

<!--
Upper 有了，現在造 Floor，也就是 τ = 0 那一端：「主體沒看過 L_ban，L_ban 列也不帶資訊」。
它來自下界模型 B0-Lower：同樣 300M、embedding 隨機初始化可訓，但只在 Multi-Train 上訓練，從沒看過 L_ban。
Floor 的定義是：拿 B0-Lower 最後的權重，把 L_ban 的列全部設成已經訓練過的列的平均向量 μ。不需要另外訓練，評估時算就好。
[click] 為什麼不直接拿 B0-Lower 原始的模型當下界？
[click] 因為 B0-Lower 的 embedding 是可訓的，而 tied 的每一列都會從 softmax 分母收到梯度，L_ban 的列從來不是正確答案，就一路被壓低，再加上 weight decay 一直縮。光看 decay 項，預設 LR 3e-3、4M batch 跑 20B，列會縮到大約 0.32 倍；batch 改成 1M，會縮到 0.006 倍。這樣的下界差得很不自然，會灌水所有的遷移比例。
平均向量的意思是「沒有任何資訊，但至少落在分布裡面」。先記住這個定義，它等一下會被改，而且結果段我們還會回來問：Floor 真的是最差的點嗎？
兩端都有了。現在回到我們的主角。
-->

---
layout: default
kicker: Setup · Embedding
clicks: 2
---

# 凍結的 E，*從哪來*？

<div class="flex flex-col gap-5 mt-4">
<div class="w-[38rem]" style="view-transition-name: setup-ematrix">
<SetupEMatrix mode="lang" :step="3" />
</div>
<div class="grid grid-cols-2 gap-5 w-[46rem]">
  <div v-click="1" class="track-card">
    <div class="track-k">軌道 1</div>
    <b>自己訓：Oracle Embedding</b>
    <span>理論上行不行</span>
  </div>
  <div v-click="2" class="track-card">
    <div class="track-k">軌道 2</div>
    <b>從開源模型拔：Donor Embedding</b>
    <span>現實中行不行</span>
  </div>
  <div v-click="2" class="col-span-2 text-[16px] text-[var(--google-muted)]">兩條軌道：E 都凍結，主體架構完全一樣</div>
</div>
</div>

<style>
.track-card {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
  padding: 1rem 1.3rem;
  border: 1px solid var(--google-border);
  border-radius: 1rem;
  background: white;
}
.track-card b { font-size: 21px; line-height: 1.3; }
.track-card span { color: var(--google-muted); font-size: 17px; }
.track-k { color: var(--google-blue-dark); font-size: 15px; font-weight: 600; }
</style>

<!--
τ 的兩端造好了，回到主角。它要一組預先對齊、凍結的 E，問題是：這組 E 從哪來？我們準備了兩條軌道，各回答一個問題。
[click] 軌道 1 是自己訓：用一個看過全部語言的模型學出 E，來源完全受控，回答「理論上行不行」。
[click] 軌道 2 是直接從開源模型拔 E，回答「現實中拿現成的 embedding 來用行不行」。兩條軌道的 E 都凍結，接的主體架構一模一樣，所以比較時只差 E 從哪來。
接下來分別看這兩組 E。
-->

---
layout: figure
kicker: 軌道 1 · 自己訓
clicks: 1
---

# Oracle Embedding

就是剛才 τ = 1 那一端的 B0-Upper：把它的 E 拔出來

::figure::

<div class="w-[38rem]">
<SetupDataBars only="oracle" />
</div>
<div class="text-[15px] text-[var(--google-muted)] -mt-1 mb-2">↓ 訓練 B0-Upper（τ = 1 那一端）</div>
<div class="w-[50rem]" style="view-transition-name: setup-ematrix">
<SetupEMatrix mode="oracle" :step="$clicks + 3" />
</div>

<!--
軌道 1 的 E 叫 Oracle Embedding。它不用另外訓練：就是剛才 τ = 1 那一端的 B0-Upper。
再把 Oracle-Train 拉回來提醒一下：Multi-Train 再「加上」中文和 Fake-French，總共約 24B tokens，所以 B0-Upper 看過中文，也看過 Fake-French。
[click] 把它訓練好的 tied embedding 拔出來，就是 Oracle_Emb：181,632 × 1024，每一列都訓練過。它是完全受控的預對齊多語 embedding：同架構、零未知的預訓練資料。
也因為它來自一個看過 L_ban 的模型，Oracle_Emb 是一個理想化的 donor，回答的是「理論上行不行」。
這裡有一個工程上的小坑：Oracle_Emb 不能從 HF 匯出的目錄拿，因為匯出時已經把 embedding 乘數乘進去了，下游再乘一次就變成平方，而且完全不會報錯。我們改從訓練的 checkpoint 拿未 fold 的權重。
-->

---
layout: default
kicker: 軌道 2 · 開源模型
---

<div class="flex items-center gap-14">
  <PaperStack :pages="['/papers/qwen3/p1.jpg','/papers/qwen3/p2.jpg','/papers/qwen3/p3.jpg']" title="Qwen Team" venue="arXiv 2025" width="14rem" vt="paper-qwen3" />
  <div class="text-left">

# Donor *Embedding*

從開源模型拔出 E，凍結，只訓主體

  </div>
</div>

<!--
軌道 2 的 E 來自外面。Donor Embedding 的意思是：把開源模型已經學好多語對齊的 embedding 矩陣拔出來，直接當新模型的輸入層和輸出層，凍結它，只訓練中間的 Transformer。
我們用的是 Qwen 團隊的 Qwen3：36T tokens、119 種語言預訓練，全系列共用 151,936 的 byte-level BPE，小模型原生就是 tied。
那要拔哪一個？下一頁。
-->

---
layout: default
kicker: 軌道 2 · 開源模型
clicks: 2
---

# 拔哪一個？要能插進*同一個主體*

<div class="absolute top-8 right-12">
  <PaperStack :pages="['/papers/qwen3/p1.jpg','/papers/qwen3/p2.jpg','/papers/qwen3/p3.jpg']" width="5rem" vt="paper-qwen3" />
</div>

<div class="grid grid-cols-2 gap-6 mt-8 w-[44rem]">
  <div class="setup-card">
    <div class="setup-card-k">候選</div>
    <b>Llama-3.2-1B</b>
    <span>d = 2048 → 主體只能疊 6 層</span>
    <div v-click="1" class="setup-confound">和軌道 1 同時差：來源、寬度、深度、參數量</div>
  </div>
  <div class="setup-card">
    <div class="setup-card-k">我們選的</div>
    <b>Qwen3-0.6B-Base</b>
    <span>d = 1024、原生 tied → 同一個 24 層主體</span>
    <div v-click="2" class="setup-clean">唯一差別：E 從哪來</div>
  </div>
</div>

<style>
.setup-card {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 1.4rem 1.5rem;
  border: 1px solid var(--google-border);
  border-radius: 1rem;
  background: white;
}
.setup-card b { font-size: 23px; line-height: 1.3; }
.setup-card span { color: var(--google-muted); font-size: 17px; line-height: 1.45; }
.setup-card-k { color: var(--google-blue-dark); font-size: 15px; font-weight: 600; }
.setup-confound { margin-top: 0.4rem; color: var(--google-red); font-size: 17px; font-weight: 600; }
.setup-clean { margin-top: 0.4rem; color: var(--google-green-dark); font-size: 17px; font-weight: 600; }
</style>

<!--
要拔哪一個開源模型？挑選的條件只有一個：拔出來的 E 要能直接插進軌道 1 那個主體，這樣兩條軌道一比，才只差「E 從哪來」。
donor 的寬度會鎖死主體的 d_model。一個自然的候選是 Llama-3.2-1B，但它寬 2048，在同樣的算力下主體只能疊 6 層。
[click] 這樣軌道 2 和軌道 1 一比，同時差了 E 的來源、寬度、深度、參數量四件事，結果出來根本分不清是哪個造成的。
[click] Qwen3-0.6B-Base 剛好原生 d=1024 而且 tied，可以放進和軌道 1 一模一樣的 24 層主體，兩者唯一的差別就是 E 從哪裡來。這也是為什麼整個實驗都用 Qwen3 的詞表；附帶好處是 Qwen3 切中文比較不碎，一個 token 大約 1.5 到 1.8 個字，V_ban 的邊界也比較乾淨。
我們也考慮過 Gemma-3-1B，但 26 萬的詞表算力稅太重：embedding 就算凍結，LM head 的 matmul 還是要算。
補充一點：軌道 2 沒有 Fake-French 的列，所以只測真中文和真法文。
兩條軌道的 E 都選好了。接下來把 E 凍結、開始訓主體，看起來就可以開跑了。真的嗎？
-->

---
layout: figure
kicker: Setup · 訓練
clicks: 4
---

# embedding 凍結了，就安全了嗎？

<div v-click="4" class="absolute top-26 right-8 z-10 flex items-start gap-3">
<div class="text-right leading-tight mt-2"><div class="text-[15px] font-600">Gao et al.</div><div class="text-[13px] text-[var(--google-muted)]">ICLR 2019</div></div>
<PaperStack :pages="['/papers/gao2019/p1.png','/papers/gao2019/p2.png','/papers/gao2019/p3.png']" width="4rem" />
</div>

::figure::

<div class="w-[46rem] mx-auto">
<SetupSoftmaxFlow :step="Math.min($clicks, 3)" />
</div>

<div class="flex items-center justify-center gap-10 -mt-1">
<div v-click="1" class="text-[20px]">

$$
\frac{\partial \mathcal{L}}{\partial h} \;=\; \sum_{v} p_v\, E_v \;-\; E_y
$$

</div>
<div v-click="3" class="text-[17px] leading-snug max-w-[27rem]">
列凍住了，能動的只有主體<br>→「被壓制」和「幾何有沒有被用上」混在一起
</div>
</div>

<!--
還記得造 Floor 時說過：B0-Lower 的 L_ban 列因為從來不是答案，被 softmax 一路壓低。那我們的主角呢？它的 embedding 是凍結的，只訓主體，可是 softmax 一樣是整個詞表。
畫面是一步訓練的全貌：輸入 museum，查 E 的那一列進主體，主體吐出 h，h 再和同一個 E 的每一列做內積，過 softmax 變成整個詞表的機率。左右兩個 E 是同一個矩陣，都凍結；E 我們轉成直的，上到下是 token ID，紅色是散落的中文列、底下綠色是 Fake-French。
[互動] 先問大家：embedding 都凍結了，L_ban 的列在訓練中完全不會動。那訓練過程還會對 L_ban 造成什麼影響嗎？
[click] 正確答案是 opens，它那一列被往上拉。看 loss 對最後一層 hidden state h 的梯度：所有 token 的機率加權平均的 E，減掉正確答案那一列。
[click] L_ban 的列永遠不是正確答案，但每一步都在這個加總裡面，等於每一步都是負例。列本身不能動，能動的只有主體，所以主體會學到讓 h 和這些列的內積變低。
[click] 更麻煩的是，L_ban 的列因為語言均值偏移本來就聚在一起，避開它們幾乎不傷 L_train。結果是：如果主實驗失敗，我們分不清是「幾何對齊沒被用上」，還是「主體被訓練成刻意避開它們」。推論時再做 mean-centering 只是事後補救，壓制已經學進主體了。
[click] 這個現象前人也看過：Gao 等人 ICLR 2019 的 representation degeneration 分析 likelihood 加 weight tying 的訓練，發現大部分 embedding 會擠進一個狹窄的錐形，低頻詞在 softmax 裡幾乎只收到「被當成負例」的梯度。我們的 L_ban 列是極端版本：不是低頻，是頻率為零，所以每一步都只當負例。
-->

---
layout: figure
kicker: Setup · 訓練
clicks: 2
---

# 修正：凍結 E，softmax 也要*一起遮*

logits 只取訓練集裡出現過的 token，其餘列不進 logsumexp

::figure::

<div class="w-[50rem]" style="view-transition-name: setup-ematrix">
<SetupEMatrix mode="mask" />
</div>

<div class="grid grid-cols-2 gap-6 w-[48rem] mt-3 text-[17px]">
  <div v-click="1"><b>遮罩只來自 token 計數</b><br><span class="text-[var(--google-muted)]">不需要知道禁的是哪個語言</span></div>
  <div v-click="2"><b class="text-[var(--google-red)]">只拿掉推力，不給拉力</b><br><span class="text-[var(--google-muted)]">不保證遷移會成功</span></div>
</div>

<!--
所以只凍結 embedding 不夠，softmax 也要一起處理。做法是訓練期 softmax 遮罩：訓練時 logits 只算在訓練集裡出現過的 token 上，其他列不進 logsumexp，也不受 weight decay。
以 Multi-Train 來說，保留 113,931 列，排除 67,701 列，裡面包含中文的 28,677 列和 Fake-French 的 29,622 列。
[click] 遮罩的定義直接來自資料管線的 Token Frequency Audit，只看計數，不需要知道我們禁的是哪個語言。
[click] 但要注意，遮罩只是把推開的力拿掉，並沒有提供任何把 h 拉向 L_ban 列的動力，所以它不保證遷移成功。
主線所有凍結 embedding 的 run 都用遮罩；完整 softmax 的版本留作消融，遮罩到底差多少，結果段會量給大家看。B0-Lower 也因此在 step 590 取消、改成遮罩版重跑，讓 Floor 跟主角在同樣的條件下訓練。
Floor 也照同樣的條件算：遮罩外的列全部設成平均值 μ，讓下界和主角站在同一個起點。
接下來還有一件事要先講清楚：一個 BPB 其實混了兩件事。
-->

---
layout: figure
kicker: Setup · 評估
clicks: 4
---

# 語言內排序 vs 語言選擇

$-\log p(y) \;=\; \underbrace{-\log p(y \mid y \in V_L)}_{\text{語言內排序}} \;\underbrace{-\,\log P(V_L)}_{\text{語言選擇}}$

::figure::

<div class="w-[48rem]">
<SetupDecomp :step="$clicks" />
</div>

<div v-click="4" class="text-[20px] font-600 text-center">
判定用的 τ：只看 <span class="text-[var(--google-red)]">語言內排序</span>
</div>

<!--
遮罩這件事也提醒我們：L_ban 的 BPB 其實混了兩件事。一件是主體願不願意把機率放到這個語言上，這會被剛才的壓制影響；另一件是在這個語言裡面，正確答案排得對不對，這才是幾何對齊。所以我們把每個 token 的 loss 拆成兩項。
[click] 先定義 V_L：該語言訓練文字裡出現過的 token 集合，包含數字、標點、EOS 這些共用錨點。
[click] 第二項是語言選擇：模型有沒有把機率放在這個語言那一組 token 上。這一項跟「主體有沒有被壓制、會不會主動說這個語言」比較有關。
[click] 第一項是語言內排序：在這一組 token 裡面，正確答案有沒有排對。這直接檢驗 embedding 的幾何有沒有被主體用上。
舉個例子，Floor 的中文幾乎全部損失都在語言內排序，4.530 加 0.063；Fake-French 則有一大塊在語言選擇，4.657 加 1.024。
[click] 上界和 Floor 都做同樣的拆解，τ 在兩項上分別算，但判定用的是語言內排序那一項。語言選擇那一欄的分母很小，數字常常跑出 0 到 1，只看正負方向。
-->

---
layout: figure
kicker: Setup · 評估
clicks: 2
---

# τ：從 Floor 走到 Upper 的*幾成*

<div class="flex items-center gap-10 -mt-1">
<div>

$$
\tau = \frac{\mathrm{BPB}_{\text{floor}} - \mathrm{BPB}_{x}}{\mathrm{BPB}_{\text{floor}} - \mathrm{BPB}_{\text{upper}}}
$$

</div>
<div class="text-[16px] leading-snug">
<div><b>分子</b>　模型 x 比沒看過好多少</div>
<div><b>分母</b>　看過與沒看過差多少</div>
<div class="text-[var(--google-muted)] mt-1">τ = 0 像沒看過 · τ = 1 像看過</div>
</div>
</div>

<div class="flex items-center gap-1.5 text-[13px] whitespace-nowrap mt-1 mb-1">
<span class="text-[var(--google-muted)] mr-1">三個 BPB 都這樣算</span>
<span class="px-2 py-0.5 rounded bg-[var(--google-surface)] border border-[var(--google-border)]">同一批 FLORES+ 1,012 句 L<sub>ban</sub></span>→
<span class="px-2 py-0.5 rounded bg-[var(--google-surface)] border border-[var(--google-border)]">逐句單獨讀 · 無上下文</span>→
<span class="px-2 py-0.5 rounded bg-[var(--google-surface)] border border-[var(--google-border)]">只計<b class="text-[var(--google-red)]">語言內排序</b></span>→
<span class="px-2 py-0.5 rounded bg-[var(--google-surface)] border border-[var(--google-border)]">Σ bits ÷ Σ bytes</span>
</div>

::figure::

<div class="mx-auto" style="width: 46rem">
<SetupTauMap :step="$clicks" />
</div>

<div class="mx-auto" style="width: 46rem; view-transition-name: tau-line">
<RevealChart
  :min="-0.2" :max="1" axis="τ（語言內排序）"
  :rows="[
    { label: '真中文', value: 0, color: 'red', at: 99 },
    { label: 'Fake-French', value: 0, color: 'green', at: 99 },
  ]"
/>
</div>

<!--
兩端的模型都造好了，現在可以把 τ 完整講一次，接下來所有結果都用它。
τ 回答的是一句話：沒看過 L_ban 的模型，追回了「看過」的幾成。分子是模型 x 比 Floor 好了多少 BPB，分母是 Floor 和 Upper 之間的整段距離。Upper 是 B0-Upper，訓練時看過 L_ban；Floor 是遮罩版 B0-Lower 把遮罩外的列設成平均值，等於完全沒看過。
所以 τ = 0 代表跟沒看過一樣，τ = 1 代表跟看過一樣，0.3 就是追回三成。它可以小於 0，也就是比 Floor 還差。
公式裡的三個 BPB 是怎麼來的？三個模型讀的是完全同一批句子：FLORES+ devtest 的 1,012 句 L_ban，每句單獨讀，前面只有一個句首 EOS，不給 prompt 也不給英文原文。每個 token 只取剛剛拆出來的語言內排序那一項，全部加起來，再除以這 1,012 句的 byte 總數。三個數字分母相同、句子相同，差別只在模型，所以可以直接相減。信賴區間也是在這 1,012 句上配對 bootstrap 出來的。
[click] 以真中文為例：Floor 是 4.530，上界是 1.457。Fake-French 對應的是 4.657 和 0.997。
[click] 這兩個端點就對到 τ 軸上的 0 和 1。下面這條 τ 數線，接下來結果的部分會一直回來。
真法文我們不放上來：它的上界 1.914 和 Floor 2.050 幾乎重合，分母太小，τ 沒有意義，真法文改看 BPB 本身和 Δ。
另外要提醒，上界和下界不只差 L_ban：上界是完整 softmax、下界是遮罩，步數也不同，22,888 對 19,073。
[互動] 在看結果之前，大家先下個注：核心實驗，也就是凍結 Oracle_Emb、主體只用 Multi-Train 訓練，中文和 Fake-French 會落在這條線的哪裡？
不過開跑之前，還有最後一件事：這把尺本身夠不夠準。
-->

---
layout: statement
kicker: 關卡
---

# 尺夠 *準* 嗎？

兩道開跑前真的可能失敗的關卡

<!--
量法有了、兩端也造好了，開跑主線之前還要問一句：這把尺夠不夠準？
原本計畫裡的 Go/No-Go 有一條是「B0-Upper 和 B0-Lower 要拉出明顯差距」，這條必然成立，擋不住任何東西。
所以我們改成真的可能失敗、而且失敗之後怎麼辦事先定好的關卡。第一道是前面看過的資料關卡，lid.176 那個 0.94%。剩下兩道：embedding 本身有沒有對齊，以及量尺的解析度夠不夠。
誠實說明一下，這兩道的判準是 11/24 才正式補寫到頁面上的，但內容取自計畫頁和評估腳本裡事先寫好的 pass 條件，沒有因為結果去調整。
-->

---
layout: figure
kicker: Setup · 關卡
clicks: 4
---

# Embedding 關卡：BLI P@1

CSLS 最近鄰；通過 = Wilson 95% CI 下界 > 打亂候選列 5 次的最大值

<div class="absolute top-6 right-12">
  <PaperStack :pages="['/papers/muse2018/p1.png','/papers/muse2018/p2.png','/papers/muse2018/p3.png']" width="4.5rem" vt="paper-muse2018" />
</div>

::figure::

<div class="w-[48rem]">
<SetupBliChart :step="$clicks" />
</div>

<div v-click="4" class="text-[16px] text-center text-[var(--google-muted)] mt-1">
MUSE 只取兩邊都是單一 token 的詞對（偏高頻）· Fake-French 是「同一語言兩套編號」，偏容易
</div>

<!--
第二道是 embedding 關卡：如果 Oracle_Emb 本身就沒有對齊，後面的實驗根本不用跑。
做法是 bilingual lexicon induction：中文用 MUSE 的 en–zh 詞典裡兩邊都是單一 token 的詞對，用 CSLS 找最近鄰、算 P@1；Fake-French 的標準答案就是平移前的原 token。通過條件是 P@1 的 Wilson CI 下界要高過打亂候選列五次的隨機基準最大值。
[click] Oracle_Emb 的 en–zh 是 0.484，隨機基準最大只有 0.00023。
[click] Fake-French 是 0.800，隨機基準 0.000034。兩個都比隨機高三個數量級以上，Go。
[click] 同一個腳本也跑了 Qwen3 0.6B 和 1.7B 當參考，都在 0.52 左右。Oracle_Emb 的 en–zh 比 Qwen3-0.6B 低大約 0.04，CI 不重疊，畢竟一個是 24B tokens、一個是 36T。
[click] 注意兩件事：MUSE 只取單 token 詞對，偏高頻詞，可能高估整個 V_ban 的對齊；Fake-French 本質上是同一個語言的兩套編號，BLI 偏容易，比較像上限參考。如果沒通過，預案是延長 B0-Upper，或主線改走 Qwen3 軌道。
還有，BLI 高只是必要條件，結果段會看到 BLI 幾乎沒掉、遷移卻歸零的例子。
-->

---
layout: figure
kicker: Setup · 關卡
clicks: 4
---

# 量尺關卡：Δ 的解析度

通過 = Upper 與 Floor 的 Δ 差距 ≥ 5 × 這個差距的 95% CI 寬度

::figure::

<div class="w-[48rem]">
<SetupRulerChart :step="$clicks" />
</div>

<div v-click="4" class="text-[19px] text-center mt-2">
CI 寬度 ≈ 上下界差距的 4–7%　→　大約能分辨 <b class="text-[var(--google-blue-dark)]">τ 差 0.05</b>
</div>

<!--
第三道是量尺關卡。動機很直接：我們要能分辨核心實驗落在 τ 0.3 還是 0.5，如果尺的刻度太粗，結果再漂亮也沒有意義。
做法是 FLORES+ 3-shot，句子層級配對 bootstrap，看上界和 Floor 的 Δ 差距，有沒有大過這個差距 CI 寬度的五倍。
[click] zh→en：Floor 0.029、上界 0.447，差距是 CI 寬度的 24.3 倍。
[click] fake→en：Floor 幾乎是 0，上界 0.645，24.0 倍。
[click] fr→en：Floor 已經有 0.202，因為真法文的 Floor 還能用共用 token，所以這把尺最窄，但也有 14.9 倍，三個都過。
[click] 換算一下，CI 寬度大約是上下界差距的 4% 到 7%，也就是大約能分辨 τ 相差 0.05 的結果。限制是只檢查了 L_ban 到英文，英文到 L_ban 的解析度沒有檢查；如果沒過，預案是改到 500M 做或縮小範圍。
好，尺準備好了：零點、滿分、刻度都定了。接下來看結果。

**在每個語言方向上，先量出 Floor 和 Upper 這兩個端點，再檢查它們之間的距離比測量雜訊大多少倍。大得夠多，這把尺才分得出 τ 0.3 和 0.5**
-->
