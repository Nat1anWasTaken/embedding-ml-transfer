---
layout: section
---

<!--
PLAN:
- 分隔頁：轉場到「還沒跑的另外兩條路」。
- 三條路線總覽：A（已做）先在，B、C 隨講者點出；每條用同一個小圖示（E / 主體 哪個凍結），讓聽眾一眼看出 B、C 是把 A 反過來。B 的圖示會 morph 到下一頁。
- 方向 B 問題頁：B 圖示＋一句話。感受：「如果允許一點 L_ban 語料呢？」
- de Vries 介紹（大 PaperStack）→ 細節（小 PaperStack 角落 morph）：decoder 版的 MonoTrans = B1 的前例。
- Active Forgetting 兩篇介紹 → 細節：重置 embedding 的時間軸示意，讓「主體學會接新 embedding」變具體。
- WECHSEL / FOCUS 介紹 → 細節：「從相近 token 組出初始值」的示意。
- B 要測什麼：空的資料效率曲線框（Floor、上界、核心實驗與核心實驗+置中兩條目標線），點擊依序加上 B1 起點與未知點、B2/B3。不放任何假數字，未知點標「?」。[互動] 在講稿。
- 方向 C 問題頁：C 圖示（E 由 H_θ 生成）＋一句話。感受：「零語料、不動主體，能不能直接生出來？」
- MIMICK / ZeTT 介紹 → 細節：ZeTT 的 Fig.1 架構圖＋Base→Instruct 的關鍵數字。
- OFA → HyperOFA 介紹 → 細節：HyperOFA pipeline 圖；它是 C2 的直接靈感。
- C 要測什麼：輸入（字形 / 對齊向量）× 語言（Fake-French / 中文）的 2×2 預期表，逐格出現；最後點出要先打敗的線性基線。
- 結尾：謝謝＋repo QR。
-->

# Other Paths

<!--
最後五分鐘，講我們計畫裡另外兩條還沒跑的路。
剛剛的 A 是凍結 embedding、訓練主體；B 和 C 都是反過來，把主體凍住，只動 embedding 這一側。
這部分沒有結果，我會講每條路在問什麼、我們從哪幾篇論文借了做法，以及我們打算怎麼測。
-->

---
layout: default
---

<div class="h-full flex flex-col justify-center">
  <div class="google-kicker mb-8">三條路線</div>
  <div class="grid grid-cols-3 gap-8 items-start">
    <div class="other-route">
      <OtherRouteGlyph route="A" vt="route-a" />
      <b>凍結 E，訓練主體</b>
      <span class="other-done">我們做完的部分</span>
    </div>
    <div v-click class="other-route">
      <OtherRouteGlyph route="B" vt="route-b" />
      <b>凍結主體，只學 L<sub>ban</sub> 的列</b>
      <span>需要 L<sub>ban</sub> 語料</span>
    </div>
    <div v-click class="other-route">
      <OtherRouteGlyph route="C" vt="route-c" />
      <b>凍結主體，H<sub>θ</sub> 生成 E</b>
      <span>零 L<sub>ban</sub> 語料</span>
    </div>
  </div>
</div>

<style>
.other-route {
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
}
.other-route b {
  margin-top: 0.6rem;
  font-size: 20px;
  font-weight: 600;
}
.other-route span {
  color: var(--google-muted);
  font-size: 16px;
}
.other-route .other-done {
  color: var(--google-blue-dark);
  font-weight: 600;
}
</style>

<!--
整個計畫有三條路。A 就是今天講的：embedding 凍結、主體訓練，主體從頭到尾沒看過 L_ban。
[click] B 把它反過來：拿一個沒看過 L_ban 的主體凍住，只學 L_ban 那些列。代價是需要一些 L_ban 語料，問題是需要多少。
[click] C 也凍結主體，但不直接學列，而是訓練一個小的 hypernetwork H_θ 去「生成」embedding，而且只在 L_train 的 token 上訓練，所以還是零 L_ban 語料。
B 和 C 用的主體都是下界模型 B0-Lower，也就是產生 Floor 的那個沒看過 L_ban 的主體，我們叫它禁語主體。
-->

---
layout: statement
kicker: 方向 B
---

<OtherRouteGlyph route="B" vt="route-b" :scale="0.9" />

# 只學 *embedding*

<p v-click>要多少 L<sub>ban</sub> 語料，才追得上 A 的零語料表現</p>

<!--
先看 B。B 的設定是：主體凍結，只解凍中文 V_ban 和 Fake-French 那些列，用 L_ban 語料去學。
[click] 它要回答的是一個很具體的量：要餵多少 L_ban 語料，才能追平核心實驗，或核心實驗加上置中的零語料表現。
如果只要很少的語料就追上，那 A 那條「完全零語料」的路，實務上的優勢就不大；如果要很多，A 的價值就更清楚。
這條路有三篇主要的前例，我們一篇一篇看。
-->

---
layout: default
---

<PaperStack :pages="['/papers/devries2021/p1.jpg','/papers/devries2021/p2.jpg','/papers/devries2021/p3.jpg']" title="de Vries & Nissim" venue="Findings of ACL 2021" width="15rem" spread="fan" vt="paper-devries2021" />

<!--
第一篇是 de Vries 和 Nissim 的 "As Good as New"，Findings of ACL 2021。
他們把英文 GPT-2 改成義大利文和荷蘭文。
做法和 Artetxe 的 MonoTrans 是同一個精神，只是換到 decoder。
-->

---
layout: default
---

<div class="absolute top-10 left-14">
  <PaperStack :pages="['/papers/devries2021/p1.jpg','/papers/devries2021/p2.jpg','/papers/devries2021/p3.jpg']" title="de Vries & Nissim" venue="Findings of ACL 2021" width="6.5rem" spread="fan" vt="paper-devries2021" />
</div>

<div class="h-full flex flex-col justify-center pl-72">
  <div class="other-idea">Transformer 層不動<br><em>只重學 lexical embedding</em></div>
  <div v-click class="other-ours">學出來的義、荷 embedding 和英文的是對齊的</div>
  <div v-click class="other-ours">→ 我們的 B1：decoder-only、tied，從 Floor 起跑</div>
</div>

<style>
.other-idea {
  font-size: 40px;
  font-weight: 700;
  line-height: 1.2;
  letter-spacing: -0.02em;
}
.other-idea em {
  color: var(--google-blue);
  font-style: normal;
}
.other-ours {
  margin-top: 1.4rem;
  color: var(--google-muted);
  font-size: 22px;
}
</style>

<!--
他們不調 Transformer 層，只重新學 lexical embedding，所以這就是 decoder-only 版的 MonoTrans。
[click] 結果是學出來的義大利文、荷蘭文 embedding 和原本的英文 embedding 是對齊的；生成的句子人還是分得出來是人造的，但評分和從頭訓練的 GPT-2 相當。
[click] 我們的 B1 就是把這件事搬到 tied 架構上：凍結禁語主體，只學 L_ban 的列。
有一個設計細節：這些列一開始設成已訓練列的平均值，也就是 Floor 本身。平均值代表「沒有資訊、但落在分布裡」，不會像範數不對的隨機列那樣在一開始造成 loss spike。
-->

---
layout: default
---

<div class="flex gap-12 items-end">
  <PaperStack :pages="['/papers/chen2023/p1.png','/papers/chen2023/p2.png','/papers/chen2023/p3.png']" title="Chen et al." venue="NeurIPS 2023" width="13rem" spread="fan" vt="paper-chen2023" />
  <PaperStack :pages="['/papers/aggarwal2025/p1.jpg','/papers/aggarwal2025/p2.jpg','/papers/aggarwal2025/p3.jpg']" title="Aggarwal et al." venue="arXiv 2024" width="13rem" spread="fan" vt="paper-aggarwal2025" />
</div>

<!--
第二組是 Active Forgetting。左邊是 Chen 等人 NeurIPS 2023 的原始版本，做在 RoBERTa 上；右邊是 Aggarwal 等人把它搬到 decoder-only LLM 的版本。
它問的是另一個角度：與其事後硬接新 embedding，能不能在預訓練時就讓主體「習慣」換 embedding？
-->

---
layout: default
---

<div class="absolute top-10 left-14 flex gap-3">
  <PaperStack :pages="['/papers/chen2023/p1.png','/papers/chen2023/p2.png','/papers/chen2023/p3.png']" title="Chen et al." venue="NeurIPS 2023" width="5rem" spread="fan" vt="paper-chen2023" />
  <PaperStack :pages="['/papers/aggarwal2025/p1.jpg','/papers/aggarwal2025/p2.jpg','/papers/aggarwal2025/p3.jpg']" title="Aggarwal et al." venue="arXiv 2024" width="5rem" spread="fan" vt="paper-aggarwal2025" />
</div>

<div class="h-full flex flex-col justify-center pl-96">
  <div class="other-idea">預訓練時<br><em>定期重置 embedding</em></div>
  <div class="other-reset mt-6" aria-hidden="true">
    <div class="seg"></div><div class="cut"></div><div class="seg"></div><div class="cut"></div><div class="seg"></div><div class="cut"></div><div class="seg"></div>
  </div>
  <div v-click class="other-ours">5M tokens 適應：XNLI 平均 +21.2%</div>
  <div v-click class="other-ours">→ B2：每 2B tokens 重置一次，再做 B1</div>
</div>

<style>
.other-idea {
  font-size: 40px;
  font-weight: 700;
  line-height: 1.2;
  letter-spacing: -0.02em;
}
.other-idea em {
  color: var(--google-blue);
  font-style: normal;
}
.other-ours {
  margin-top: 1.4rem;
  color: var(--google-muted);
  font-size: 22px;
}
.other-reset {
  display: flex;
  align-items: center;
  width: 26rem;
  height: 1.1rem;
}
.other-reset .seg {
  flex: 1;
  height: 0.55rem;
  border-radius: 0.3rem;
  background: var(--google-blue-soft);
  border: 1px solid var(--google-blue);
}
.other-reset .cut {
  width: 3px;
  height: 1.6rem;
  margin: 0 0.35rem;
  background: var(--google-red);
}
</style>

<!--
做法很直接：預訓練的時候，每隔 K 步就把 embedding 層重置成隨機的，主體繼續訓練。效果像 meta-learning，主體被逼著學會「快速接上一組新的 embedding」。
[click] Chen 等人的結果是，適應語料只有 5M tokens 時，XNLI 平均增益 21.2%，而且對阿拉伯文、泰文這類離英文遠的語言幫助最大。Aggarwal 搬到 decoder 上，7 個多語 benchmark 勝出 6 個，表徵也更 isotropic。
[click] 我們的 B2 就是重跑一條 300M × 20B 的主體，每 2B tokens 重置一次 embed_tokens 和它的 AdamW state，然後做跟 B1 一模一樣的適應，直接比主體的可塑性。
在 tied 架構下要注意，重置 embedding 等於同時重置 output head，每次重置後 loss 會跳回大約 ln V。Aggarwal 也提醒我們，除了 L_ban 學多快，還要盯著 L_train 有沒有被忘掉。
-->

---
layout: default
---

<div class="flex gap-12 items-end">
  <PaperStack :pages="['/papers/wechsel2022/p1.jpg','/papers/wechsel2022/p2.jpg','/papers/wechsel2022/p3.jpg']" title="WECHSEL · Minixhofer et al." venue="NAACL 2022" width="13rem" spread="fan" vt="paper-wechsel2022" />
  <PaperStack :pages="['/papers/focus2023/p1.jpg','/papers/focus2023/p2.jpg','/papers/focus2023/p3.jpg']" title="FOCUS · Dobler & de Melo" venue="EMNLP 2023" width="13rem" spread="fan" vt="paper-focus2023" />
</div>

<!--
第三組是初始化的捷徑：WECHSEL，NAACL 2022；FOCUS，EMNLP 2023。
B1 從平均值起跑，等於什麼資訊都不給；這兩篇問的是，如果一開始就用語意上相近的 token 組出初始值，能省多少訓練。
-->

---
layout: default
---

<div class="absolute top-10 left-14 flex gap-3">
  <PaperStack :pages="['/papers/wechsel2022/p1.jpg','/papers/wechsel2022/p2.jpg','/papers/wechsel2022/p3.jpg']" title="WECHSEL" venue="NAACL 2022" width="5rem" spread="fan" vt="paper-wechsel2022" />
  <PaperStack :pages="['/papers/focus2023/p1.jpg','/papers/focus2023/p2.jpg','/papers/focus2023/p3.jpg']" title="FOCUS" venue="EMNLP 2023" width="5rem" spread="fan" vt="paper-focus2023" />
</div>

<div class="h-full flex flex-col justify-center pl-96">
  <div class="other-idea">新 token 的初始值<br>= <em>相近舊 token 的組合</em></div>
  <div v-click class="other-ours">WECHSEL：訓練量最多少 64 倍，就勝過從頭訓練</div>
  <div v-click class="other-ours">→ B3：用這種初始化，測 0 步與 100M tokens</div>
</div>

<style>
.other-idea {
  font-size: 40px;
  font-weight: 700;
  line-height: 1.2;
  letter-spacing: -0.02em;
}
.other-idea em {
  color: var(--google-blue);
  font-style: normal;
}
.other-ours {
  margin-top: 1.4rem;
  color: var(--google-muted);
  font-size: 22px;
}
</style>

<!--
兩篇的核心想法一樣：新 token 的 embedding，用舊詞表裡語意相近的 token 組合出來。WECHSEL 用對齊好的 fastText 靜態詞向量找相近的英文 token；FOCUS 不需要雙語詞典，用 sparsemax 對新舊詞表重疊的 token 做稀疏組合。
[click] WECHSEL 的結果是訓練量最多少 64 倍，就能勝過同大小、從頭訓練的模型；FOCUS 在 XLM-R 上 LM、NLI、QA、NER 都勝過隨機初始化。
[click] 我們的 B3 用這類初始化填禁語主體裡的 L_ban 列，先直接測 0 步，再跑 100M tokens 輕量微調。
B3、B1 和核心實驗剛好是一個三角：B1 平均值起跑、不帶資訊；B3 帶相似度資訊起跑；核心實驗是凍結預對齊、零語料。
-->

---
layout: default
---

<div class="h-full flex flex-col justify-center">
  <div class="google-kicker mb-2">方向 B 要畫的那張圖</div>
  <svg viewBox="0 0 780 360" class="w-full" style="max-height: 24rem">
    <line x1="90" y1="310" x2="740" y2="310" stroke="#5f6368" stroke-width="1.5" />
    <line x1="90" y1="40" x2="90" y2="310" stroke="#5f6368" stroke-width="1.5" />
    <text x="415" y="350" text-anchor="middle" style="font-size: 16px" fill="#5f6368">L_ban 適應語料（tokens）</text>
    <text x="30" y="175" text-anchor="middle" style="font-size: 16px" fill="#5f6368" transform="rotate(-90 30 175)">τ（真中文，整體 BPB）</text>
    <g style="font-size: 15px" fill="#5f6368" text-anchor="middle">
      <text x="120" y="332">0</text>
      <text x="330" y="332">100M</text>
      <text x="520" y="332">500M</text>
      <text x="700" y="332">2B</text>
    </g>
    <line x1="90" y1="50" x2="740" y2="50" stroke="#9aa0a6" stroke-dasharray="5 5" />
    <text x="745" y="54" style="font-size: 14px" fill="#5f6368">上界 1</text>
    <text x="745" y="314" style="font-size: 14px" fill="#5f6368">Floor 0</text>
    <g v-click>
      <line x1="90" y1="230.2" x2="740" y2="230.2" stroke="#4285f4" stroke-width="2.5" />
      <text x="96" y="222" style="font-size: 15px" fill="#1967d2" font-weight="600">方向 A 零語料 0.307</text>
      <line x1="90" y1="188.6" x2="740" y2="188.6" stroke="#4285f4" stroke-width="2.5" stroke-dasharray="8 5" />
      <text x="96" y="180" style="font-size: 15px" fill="#1967d2" font-weight="600">方向 A + 推論期置中 0.467</text>
    </g>
    <g v-click>
      <circle cx="120" cy="310" r="8" fill="#202124" />
      <text x="136" y="298" style="font-size: 15px" fill="#202124" font-weight="600">B1 起點 = Floor</text>
      <g style="font-size: 20px" font-weight="700" fill="#5f6368" text-anchor="middle">
        <circle cx="330" cy="265" r="14" fill="white" stroke="#202124" stroke-width="2" stroke-dasharray="3 3" />
        <text x="330" y="272">?</text>
        <circle cx="520" cy="265" r="14" fill="white" stroke="#202124" stroke-width="2" stroke-dasharray="3 3" />
        <text x="520" y="272">?</text>
        <circle cx="700" cy="265" r="14" fill="white" stroke="#202124" stroke-width="2" stroke-dasharray="3 3" />
        <text x="700" y="272">?</text>
      </g>
    </g>
    <g v-click style="font-size: 15px" font-weight="600">
      <rect x="430" y="80" width="290" height="62" rx="8" fill="#f8f9fa" stroke="#dadce0" />
      <text x="444" y="105" fill="#34a853">B2：換成 Active Forgetting 主體</text>
      <text x="444" y="130" fill="#ea4335">B3：0 步就帶相似度初始化</text>
    </g>
  </svg>
</div>

<!--
這是 B 最後要畫出來的圖。橫軸是 L_ban 適應語料量，100M、500M、2B 三個點；縱軸是真中文的 τ，0 是 Floor，1 是上界。
[click] 先把目標線畫上去：核心實驗完全零語料，整體 BPB 的 τ 是 0.307，再加上推論期置中是 0.467。這兩條是 B 要追的線。
[click] B1 的起點在定義上就是 Floor，因為它的 L_ban 列一開始就是平均值；之後三個語料量的點，我們還不知道。
[互動] 大家可以先想一下：一個凍結的、沒看過中文的主體，只學中文那幾萬列，要多少語料才會穿過 0.307 那條線？100M 就夠，還是 2B 都不夠？
[click] B2 是同一條曲線但換成 Active Forgetting 的主體，看整條曲線有沒有往上移；B3 則看 0 步的起點能不能直接離開 Floor。
實作上有一個要注意的地方：100M tokens 在大 batch 下只有幾十步，所以適應訓練要用小 batch，三個語料量固定 batch 和 warmup，讓曲線只差在 token 數。
-->

---
layout: statement
kicker: 方向 C
---

<OtherRouteGlyph route="C" vt="route-c" :scale="0.9" />

# *生成* embedding

<p v-click>主體不動、零 L<sub>ban</sub> 語料</p>

<!--
接著是 C。C 的想法是訓練一個輕量的 hypernetwork H_θ，輸入某種外部表徵，輸出凍結主體看得懂的 tied embedding 列。
[click] 它想同時守住兩個限制：主體不動，而且完全不用 L_ban 語料。H_θ 只在 L_train 的 token 上學，推論時把 L_ban 的 token 丟進去，零樣本生成它們的列。
如果做得到，它就同時繞過 A 的空間失配，和 B 對語料的依賴。
這條路的前例也是三篇。
-->

---
layout: default
---

<div class="flex gap-12 items-end">
  <PaperStack :pages="['/papers/mimick2017/p1.jpg','/papers/mimick2017/p2.jpg','/papers/mimick2017/p3.jpg']" title="MIMICK · Pinter et al." venue="EMNLP 2017" width="13rem" spread="fan" vt="paper-mimick2017" />
  <PaperStack :pages="['/papers/zett2024/p1.png','/papers/zett2024/p2.png','/papers/zett2024/p3.png']" title="ZeTT · Minixhofer et al." venue="NeurIPS 2024" width="13rem" spread="fan" vt="paper-zett2024" />
</div>

<!--
左邊是 MIMICK，EMNLP 2017：在詞表層級訓練一個字元 Bi-LSTM，從拼寫預測現有的詞向量，拿來零樣本生成 OOV 詞的 embedding。
右邊是 ZeTT，Zero-Shot Tokenizer Transfer，NeurIPS 2024，可以看成是 MIMICK 的現代、大規模版本。
-->

---
layout: default
---

<div class="absolute top-10 left-14 flex gap-3">
  <PaperStack :pages="['/papers/mimick2017/p1.jpg','/papers/mimick2017/p2.jpg','/papers/mimick2017/p3.jpg']" title="MIMICK" venue="EMNLP 2017" width="5rem" spread="fan" vt="paper-mimick2017" />
  <PaperStack :pages="['/papers/zett2024/p1.png','/papers/zett2024/p2.png','/papers/zett2024/p3.png']" title="ZeTT" venue="NeurIPS 2024" width="5rem" spread="fan" vt="paper-zett2024" />
</div>

<div class="h-full flex flex-col justify-center pl-96">
  <img src="/papers/zett2024/fig.png" class="other-fig" alt="ZeTT Fig. 1: tokenizer → hypernetwork → embeddings">
  <div v-click class="other-ours">Base 上練的 H<sub>θ</sub> 直接套 Instruct：MT-Bench 差約 0.5 內</div>
  <div v-click class="other-ours">→ C1：從 token 的 byte 序列預測禁語主體的列</div>
</div>

<style>
.other-fig {
  max-width: 30rem;
  max-height: 14rem;
  border-radius: 6px;
  background: white;
  box-shadow: 0 4px 18px rgba(60, 64, 67, 0.12);
}
.other-ours {
  margin-top: 1.2rem;
  color: var(--google-muted);
  font-size: 21px;
}
</style>

<!--
這是 ZeTT 的架構：給一個 tokenizer，hypernetwork H_θ 預測整套 input 和 output embedding。訓練時動態抽樣不同的 Unigram tokenizer，再加一個重建舊詞表的 MSE 輔助 loss。在 XLM-R 和 Mistral-7B 上都接近原模型，剩下的差距用不到 1B tokens 的 continued training 就補上。
[click] 最有意思的是這個結果：在 Base 上訓練的 hypernetwork 可以直接套到 Instruct 版，Mistral-7B-Instruct 的 MT-Bench 和原版差約 0.5 以內。
[click] 我們的 C1 簡化成 MIMICK 式：輸入每個 token 的 byte 序列，在 L_train 的 token 上學著預測禁語主體的列。因為我們是 tied，ZeTT 的兩個輸出頭可以合成一個。
但它有一個註定的弱點：中文的 UTF-8 lead byte 在 Multi-Train 裡幾乎沒出現，所以 C1 在中文上失敗是設計決定的，它主要是對照組。
-->

---
layout: default
---

<div class="flex gap-12 items-end">
  <PaperStack :pages="['/papers/ofa2024/p1.jpg','/papers/ofa2024/p2.jpg','/papers/ofa2024/p3.jpg']" title="OFA · Liu et al." venue="Findings of NAACL 2024" width="10rem" spread="fan" vt="paper-ofa2024" />
  <PaperStack :pages="['/papers/hyperofa2025/p1.jpg','/papers/hyperofa2025/p2.jpg','/papers/hyperofa2025/p3.jpg']" title="HyperOFA · Özeren et al." venue="arXiv 2025" width="15rem" spread="fan" vt="paper-hyperofa2025" />
</div>

<!--
第三篇是 HyperOFA，左邊小的是它的前身 OFA，Findings of NAACL 2024。
OFA 用外部對齊好的多語靜態詞向量，把新 subword 的 embedding 初始化成舊 embedding 的凸組合，有點像剛剛 B3 那一類。
HyperOFA 把這個凸組合換成一個 hypernetwork，剛好是 B 和 C 之間的橋。
-->

---
layout: default
---

<div class="absolute top-10 left-14">
  <PaperStack :pages="['/papers/hyperofa2025/p1.jpg','/papers/hyperofa2025/p2.jpg','/papers/hyperofa2025/p3.jpg']" title="HyperOFA" venue="arXiv 2025" width="6.5rem" spread="fan" vt="paper-hyperofa2025" />
</div>

<div class="h-full flex flex-col justify-center pl-72">
  <img src="/papers/hyperofa2025/fig.png" class="other-fig" alt="HyperOFA Fig. 1: external word vectors → hypernetwork → target embeddings">
  <div v-click class="other-ours">22 種語言都勝過隨機初始化，不輸、甚至勝過 OFA</div>
  <div v-click class="other-ours">→ C2：輸入換成對齊向量，只在 L<sub>train</sub> 上訓練</div>
</div>

<style>
.other-fig {
  max-width: 34rem;
  max-height: 15rem;
  border-radius: 6px;
  background: white;
  box-shadow: 0 4px 18px rgba(60, 64, 67, 0.12);
}
.other-ours {
  margin-top: 1.2rem;
  color: var(--google-muted);
  font-size: 21px;
}
</style>

<!--
HyperOFA 的 pipeline：用 source-language 的 token 訓練一個 BiLSTM hypernetwork，把外部多語詞向量空間映射到預訓練模型的 embedding 空間，再替 target-language 的 token 生成 embedding。
[click] 在 22 種高、中、低資源語言上一致勝過隨機初始化，continued pretraining 的收斂和下游表現也不輸、甚至勝過 OFA。
[click] 這是 C2 的關鍵靈感：輸入不再是字形，而是預先對齊的外部向量，這樣中文這種異字元、沒有拼寫線索的語言也有機會。C2 只在 L_train 的 token 上訓練 H_θ，推論時零樣本生成 L_ban 的 tied embedding。
外部向量分兩組：一組用我們自己的 Oracle_Emb，當上限；一組用 BGE-M3 或 fastText 這種真實的外部多語向量，當實戰版本。
-->

---
layout: default
---

<div class="h-full flex flex-col justify-center">
  <div class="google-kicker mb-5">方向 C 要測什麼</div>
  <div class="other-grid">
    <div></div>
    <div class="hd">Fake-French</div>
    <div class="hd">真中文</div>
    <div class="rh"><b>C1</b><span>輸入：byte 序列</span></div>
    <div v-click class="cell">拼寫可拆<br><em>預期接近真法文</em></div>
    <div v-click class="cell bad">lead byte 幾乎沒出現<br><em>設計上就會失敗</em></div>
    <div class="rh"><b>C2</b><span>輸入：對齊向量</span></div>
    <div v-click class="cell good c2" style="grid-column: span 2">只在 L<sub>train</sub> 上學映射 → 零樣本生成 L<sub>ban</sub> 列<br><em>目標：在中文上贏過 C1，逼近核心實驗</em></div>
  </div>
  <div v-click class="other-baseline">要先打敗：L<sub>train</sub> 上的線性 / Procrustes 映射（不花 GPU）</div>
</div>

<style>
.other-grid {
  display: grid;
  grid-template-columns: 12rem 1fr 1fr;
  gap: 0.8rem;
}
.other-grid .hd {
  color: var(--google-muted);
  font-size: 18px;
  font-weight: 600;
  text-align: center;
}
.other-grid .rh {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
.other-grid .rh b {
  color: var(--google-blue-dark);
  font-size: 28px;
}
.other-grid .rh span {
  color: var(--google-muted);
  font-size: 16px;
}
.other-grid .cell {
  padding: 0.9rem 1rem;
  border: 1px solid var(--google-border);
  border-radius: 0.7rem;
  background: var(--google-surface);
  font-size: 19px;
  line-height: 1.35;
  text-align: center;
}
.other-grid .cell em {
  color: var(--google-muted);
  font-style: normal;
  font-size: 16px;
}
.other-grid .cell.bad {
  border-color: var(--google-red);
}
.other-grid .cell.good {
  border-color: var(--google-green);
  background: #e6f4ea;
}
.other-baseline {
  margin-top: 1.4rem;
  padding: 0.7rem 1rem;
  border-left: 4px solid var(--google-yellow);
  font-size: 20px;
  font-weight: 600;
}
</style>

<!--
把 C 的實驗排成一張表。列是 hypernetwork 的輸入，欄是兩個 L_ban 條件。
[click] C1 吃 byte 序列：Fake-French 的 token 讀的是它原本的法文拼寫，拼寫可以拆，所以預期會接近真法文。
[click] 但中文的 byte 在訓練語料裡幾乎沒出現，C1 在中文上失敗是設計本身決定的。
[click] C2 改吃對齊向量，只在 L_train 上學映射。我們最想知道的是它在中文上能不能贏過 C1，甚至逼近核心實驗。
[互動] 想一下，C2 第一組的輸入就是 Oracle_Emb，而核心實驗告訴我們 Oracle_Emb 的幾何主體是能用的；那 H_θ 學到的映射，會不會其實就只是個線性變換？
[click] 所以我們會先放一個不花 GPU 的基線：用 L_train 上的配對做最小平方或正交 Procrustes，再套到 V_ban。兩邊是同一個 stack、同寬度，映射很可能接近線性，hypernetwork 要贏過它才有意義。
另外還有一組 C3a，用 Qwen3 本身量 hypernetwork 的重建保真度，以及 Base 到 Instruct 能不能直接套用，仿照 ZeTT。成本上要注意，訓練 H_θ 時梯度要穿過整個凍結主體，每個 token 大約是一般訓練的三分之二。
-->

---
layout: end
---

# 謝謝

Q&amp;A

<!--
今天就分享到這裡。總結一句：embedding 的幾何比我們想像中有用許多，一個沒看過某語言的主體，只靠放對位置的列就能讀懂它；不過目前所有證據都還停在 likelihood。
B 和 C 是接下來想跑的兩條路，一條問要多少語料，一條問能不能完全不用語料。
謝謝大家，有問題歡迎提出來。
-->
