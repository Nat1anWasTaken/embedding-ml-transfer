---
layout: section
---

<!--
PLAN（整段只有一個主張：Embedding 幾何比想像中的還要有用許多）
- 分隔頁：聽眾剛聽完八個實驗，需要喘口氣；畫面只有「Conclusion」，讓大家知道要收斂了。
- 主張頁：聽到「我們只想帶走一句話」；畫面只有那一句，其他都不放，避免跟講者搶。感受：這是整場的 takeaway。主張文字縮到下一頁頂端（view-transition concl-thesis）。
- 想像中：聽到「比想像中」的「想像」是誰的想像；看 Artetxe 2020 對 CLWE 對照組的評語（intro 看過的同一句），點擊後出現我們自己開跑前的預期。感受：原本的預期很低，為下一頁的反差鋪墊。
- 證據：主張留在頂端；三列依講者順序點出，每列一個主張＋一個大數字＋實驗標記（沿用 Results 的名稱）：讀得懂（τ）、靠的是幾何（打亂後 Δ 消失）、差的多半只是位置（置中、Qwen3）。感受：三個獨立角度都指向同一句話。
- 如果是真的：聽到推論（不是已證明的結論）；看一句開放問題，再點出「列從哪來」的 A／B／C 三種答案，B、C 就是下一段 Other Paths。感受：好奇，想知道 B、C 怎麼做。
- Caveats：使用者指定的條列；每點幾個字、依序出現，細節在講稿。感受：知道這句話在什麼範圍內成立。最後一句話接到 Other Paths。
-->

# Conclusion

<!--
好，八個實驗講完了，我們來收斂。
剛剛的結果很多，每一個實驗都有自己的小結論，但我們不打算在這裡一條一條重複。我們只想讓大家帶走一句話，接下來三張投影片都是在講這一句話為什麼站得住；最後再講它的邊界在哪裡。
-->

---
layout: statement
kicker: 如果只帶走一句話
---

<h1 style="view-transition-name: concl-thesis">Embedding 幾何<br>比想像中的還要<em>有用許多</em></h1>

<!--
這句話就是：Embedding 幾何比想像中的還要有用許多。
這裡講的「幾何」，是指 embedding 裡每一列放在哪裡、列跟列之間的相對位置；講的「有用」，是指一個沒看過某個語言的 Transformer 主體，光靠這個幾何就能用上那個語言。
那「想像中」是誰的想像？我們先回頭看一下，大家原本對這件事的預期有多低。
-->

---
layout: default
clicks: 1
---

<div class="h-full flex flex-col justify-center">
  <div class="google-kicker mb-8">想像中</div>
  <div class="grid grid-cols-[11rem_1fr] items-center gap-10">
    <PaperStack :pages="['/papers/artetxe2020/p1.jpg','/papers/artetxe2020/p2.jpg','/papers/artetxe2020/p3.jpg']" title="Artetxe et al." venue="ACL 2020" width="9rem" vt="concl-artetxe" />
    <div>
      <div class="concl-quote">“mapping between two fixed embedding spaces may be <em>overly restrictive</em>”</div>
      <div class="concl-quote-src">對 baseline CLWE 的評語：凍結的靜態詞向量 + BERT encoder</div>
    </div>
  </div>
  <div v-click="1" class="concl-ours">
    <b>我們自己開跑前</b><span>預期核心實驗會<em>失敗</em></span>
  </div>
</div>

<style>
.concl-quote {
  font-size: 32px;
  font-weight: 600;
  line-height: 1.25;
  letter-spacing: -0.01em;
}
.concl-quote em {
  color: var(--google-blue);
  font-style: normal;
}
.concl-quote-src {
  margin-top: 0.9rem;
  color: var(--google-muted);
  font-size: 17px;
}
.concl-ours {
  display: flex;
  align-items: baseline;
  gap: 1.2rem;
  margin-top: 2.6rem;
  padding-left: 1rem;
  border-left: 4px solid var(--google-gray);
}
.concl-ours b {
  color: var(--google-muted);
  font-size: 20px;
  font-weight: 600;
}
.concl-ours span {
  font-size: 30px;
  font-weight: 700;
}
.concl-ours em {
  color: var(--google-red);
  font-style: normal;
}
</style>

<!--
第一個是文獻上的預期。Intro 看過這句：Artetxe 他們在 ACL 2020 說，在兩個固定的 embedding 空間之間做映射，may be overly restrictive。
再提醒一次這句話的脈絡：這是他們對 baseline CLWE 的評語，不是整篇論文的結論；那個 CLWE 用的是事後對齊的靜態詞向量，配 BERT encoder 做下游任務。他們的主方法 MonoTrans 是反方向的，凍結主體、重學 embedding，這個等一下 Other Paths 會再看到。
[click] 第二個是我們自己的預期。開跑之前，我們其實預期核心實驗會失敗：新 token 的 τ 應該接近 0，因為 L_ban 的列會因為語言均值偏移聚在一起，主體只要學會避開那一個方向，整個語言就被壓下去了。
結果跟這兩個預期都相反。下一張用三個角度說明。
-->

---
layout: default
clicks: 3
---

<div class="concl-thesis-mini" style="view-transition-name: concl-thesis">Embedding 幾何比想像中的還要<em>有用許多</em></div>

<div class="concl-ev">
  <div v-click="1" class="concl-ev-row">
    <div class="concl-ev-claim">
      <span class="n">1</span>
      <div>
        <b>沒看過，也<em>讀得懂</em></b>
        <ResultsExpTag exp="core" :corner="false" />
      </div>
    </div>
    <div class="concl-ev-num">
      <div class="big">0.297 <small>/</small> 0.436</div>
      <div class="unit">τ · 中文 / Fake-French · Floor = 0</div>
    </div>
    <div class="concl-ev-note">翻譯增益 Δ − Δ<sub>floor</sub><br>+0.242 / +0.386</div>
  </div>
  <div v-click="2" class="concl-ev-row">
    <div class="concl-ev-claim">
      <span class="n">2</span>
      <div>
        <b>靠的是<em>幾何</em>本身</b>
        <ResultsExpTag :exp="['core', 'a4a']" :corner="false" />
      </div>
    </div>
    <div class="concl-ev-num">
      <div class="big">0.270 <small>→</small> 0.029</div>
      <div class="unit">打亂列的對應 · 中文 → En 翻譯增益 Δ</div>
    </div>
    <div class="concl-ev-note">對齊已崩潰的模型<br>打亂前後只差 +0.0001</div>
  </div>
  <div v-click="3" class="concl-ev-row">
    <div class="concl-ev-claim">
      <span class="n">3</span>
      <div>
        <b>差的多半只是<em>位置</em></b>
        <ResultsExpTag :exp="['center', 'qwen']" :corner="false" />
      </div>
    </div>
    <div class="concl-ev-num">
      <div class="big">3.632 <small>→</small> 3.131</div>
      <div class="unit">推論期置中 · 零訓練 · 中文 BPB</div>
    </div>
    <div class="concl-ev-note">Qwen3 donor 與核心實驗<br>差距 0.77 → 0.11</div>
  </div>
</div>

<style>
.concl-thesis-mini {
  color: var(--google-muted);
  font-size: 22px;
  font-weight: 700;
}
.concl-thesis-mini em {
  color: var(--google-blue);
  font-style: normal;
}
.concl-ev {
  display: flex;
  flex-direction: column;
  gap: 1.1rem;
  margin-top: 1.6rem;
}
.concl-ev-row {
  display: grid;
  grid-template-columns: 16.5rem 1fr 11.5rem;
  align-items: center;
  gap: 1.4rem;
  padding: 0.95rem 1.2rem;
  border: 1px solid var(--google-border);
  border-radius: 14px;
  background: var(--google-surface);
}
.concl-ev-claim {
  display: flex;
  align-items: flex-start;
  gap: 0.8rem;
}
.concl-ev-claim .n {
  display: inline-grid;
  flex: none;
  place-items: center;
  width: 1.8rem;
  height: 1.8rem;
  border-radius: 50%;
  background: var(--google-blue);
  color: white;
  font-size: 15px;
  font-weight: 700;
}
.concl-ev-claim b {
  display: block;
  margin-bottom: 0.45rem;
  font-size: 22px;
  font-weight: 700;
  line-height: 1.25;
}
.concl-ev-claim em {
  color: var(--google-blue);
  font-style: normal;
}
/* The badges here only label rows; they must not steal the Results morph names. */
.concl-ev .exp-mini {
  view-transition-name: none !important;
}
.concl-ev-num .big {
  color: var(--google-ink);
  font-size: 40px;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  line-height: 1.1;
  letter-spacing: -0.02em;
}
.concl-ev-num .big small {
  color: var(--google-gray);
  font-size: 28px;
  font-weight: 500;
}
.concl-ev-num .unit {
  margin-top: 0.3rem;
  color: var(--google-muted);
  font-size: 14px;
  white-space: nowrap;
}
.concl-ev-note {
  color: var(--google-muted);
  font-size: 15px;
  line-height: 1.4;
}
</style>

<!--
主張留在上面，下面是三個獨立的角度，用的都是 Results 看過的實驗。
[click] 第一，沒看過的語言，主體讀得懂。核心實驗的主體從頭到尾沒看過中文和 Fake-French，只是把凍結、預對齊的那些列放回去，語言內排序的 τ 就是中文 0.297、Fake-French 0.436，明顯離開 Floor；翻譯增益扣掉 Floor 之後也是 +0.242 和 +0.386，兩個 seed 結論一樣。
[click] 第二，靠的是幾何本身，不是「多了一些不同的向量」而已。同一個 checkpoint，把 L_ban 的列在組內互換，向量都還在，只是 token 對錯了列，中文翻英文的翻譯增益就從 0.270 掉到 0.029，Fake-French 從 0.388 掉到 0.003。反過來看，全部可訓那個對齊已經崩潰的模型，打亂前後的中文語言內 BPB 只差 +0.0001：沒有幾何可以破壞，打亂就沒差。所以有沒有用，取決於列有沒有放在對的位置。
[click] 第三，語言之間剩下的差距，有很大一塊只是一個整體的位置偏移。推論期置中完全不重訓，只把 L_ban 的列整群平移到 L_train 的均值，核心實驗的中文 BPB 就從 3.632 降到 3.131。更有意思的是 Qwen3 donor：直接借一個開源模型的 embedding，原始中文 BPB 比核心實驗差 0.77，兩個都置中之後只差大約 0.11。也就是說，別人訓練出來的幾何，我們的主體一樣用得上，差的主要是位置。
三個角度合起來，就是這句話：幾何本身帶了比我們預期多很多的資訊。
-->

---
layout: default
clicks: 2
---

<div class="h-full flex flex-col justify-center">
  <div class="google-kicker mb-4">如果這是真的</div>
  <h2 class="concl-q">一個語言的入場券，也許是<br><em>embedding 裡放對位置的列</em>，<br>而不是它的訓練資料？</h2>
  <div class="concl-routes">
    <div class="concl-route">
      <OtherRouteGlyph route="A" :scale="0.7" />
      <b>借來</b>
      <span>今天：Oracle / Qwen3 的列</span>
    </div>
    <div v-click="1" class="concl-route">
      <OtherRouteGlyph route="B" :scale="0.7" />
      <b>學出來</b>
      <span>要多少 L<sub>ban</sub> 語料？</span>
    </div>
    <div v-click="2" class="concl-route">
      <OtherRouteGlyph route="C" :scale="0.7" />
      <b>生成出來</b>
      <span>不看 L<sub>ban</sub> 能做到嗎？</span>
    </div>
  </div>
</div>

<style>
.concl-q {
  margin: 0;
  font-size: 34px;
  font-weight: 700;
  line-height: 1.3;
  letter-spacing: -0.01em;
}
.concl-q em {
  color: var(--google-blue);
  font-style: normal;
}
.concl-routes {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.6rem;
  margin-top: 2.4rem;
}
.concl-route {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
}
.concl-route b {
  margin-top: 0.5rem;
  font-size: 22px;
  font-weight: 700;
}
.concl-route span {
  color: var(--google-muted);
  font-size: 16px;
}
</style>

<!--
如果這句話成立，它會帶出一個更大的問題。這裡要講清楚：下面是推論和開放問題，不是我們已經證明的事。
畫面上這句：一個語言的入場券，也許是 embedding 裡放對位置的那些列，而不是那個語言的訓練資料？至少在 likelihood 這個層面，核心實驗的主體一句中文都沒看過，只要列放對了，它就能用。
那下一個問題自然是：這些「放對位置的列」要從哪裡來？
今天的 A 路線是借來的：Oracle 或 Qwen3，都是一個已經看過這個語言的模型給的。
[click] 第二種是學出來：主體凍住，只用一點點 L_ban 語料去學那些列。問題是，到底要多少語料才夠？
[click] 第三種是生成出來：訓練一個 hypernetwork，只在 L_train 的 token 上學「怎麼生出一列」，再拿去生 L_ban 的列，完全不看 L_ban。
這兩條就是我們計畫裡還沒跑的 B 和 C，最後一段會講。在那之前，先把今天這句話的邊界講清楚。
-->

---
layout: default
clicks: 8
---

# Caveats

<ul class="concl-cav">
  <li v-click="1"><b>只量 likelihood</b>：還沒有 XNLI、chrF++、off-target rate</li>
  <li v-click="2"><b>單一尺度</b>：300M × 20B tokens</li>
  <li v-click="3"><b>多數 run 只有一個 seed</b>；bootstrap CI 不含訓練隨機性</li>
  <li v-click="4"><b>Oracle embedding 是理想 donor</b>：它自己看過 L<sub>ban</sub></li>
  <li v-click="5"><b>推論期置中用了 L<sub>ban</sub> 的 token 統計</b>：不是嚴格零語料</li>
  <li v-click="6"><b>Fake-French 是造出來的</b>：法文的 token ID 平移</li>
  <li v-click="7"><b>均值偏移是一大塊，不是全部</b>：置中後中文 3.131，上界 1.459</li>
  <li v-click="8"><b>詞彙重疊仍遠勝對齊</b>：Fake-French 比真法文差 2.1 bits/byte</li>
</ul>

<style>
.concl-cav {
  margin-top: 1.4rem;
  padding-left: 1.3rem;
  font-size: 21px;
  line-height: 1.35;
}
.concl-cav li {
  margin-bottom: 0.62rem;
  color: var(--google-muted);
}
.concl-cav li::marker {
  color: var(--google-yellow);
}
.concl-cav b {
  color: var(--google-ink);
  font-weight: 600;
}
</style>

<!--
最後是 caveats，這些是這句話目前還不能延伸到的地方。
[click] 第一，所有判定都只看 likelihood：BPB 和 teacher-forced 的翻譯增益。我們沒有真的讓模型生成、沒有量 off-target rate，XNLI 和 chrF++ 也都還沒做。所以「讀得懂」「生得出來」都是 likelihood 層面的說法；這也是為什麼不能直接拿來跟 Artetxe 的 XNLI 數字比。
[click] 第二，只有 300M、20B tokens 一個尺度，放大之後這個現象會變強還是變弱，我們不知道。
[click] 第三，核心實驗和只動 L_train 列有兩個 seed，其他像純英文主體、Qwen3 donor、解綁輸出頭都只有一個 seed。而且 bootstrap CI 只算評估句子的抽樣誤差，不含訓練隨機性：核心實驗兩個 seed 之間，Fake-French 語言內 BPB 就差 0.123。所以單一 seed 的小差距不能下結論。
[click] 第四，Oracle embedding 是從看過中文和 Fake-French 的上界模型拔出來的，是最理想的 donor。真實情況比較接近 Qwen3 donor：幾何一樣用得上，但原始 BPB 差 0.77，要置中才救回來。
[click] 第五，推論期置中雖然零訓練，但 L_ban 的均值是用 L_ban 語料的 token 頻率加權算的，跟評估集分開，可是嚴格來說不是零目標語料。完全不用 L_ban 文字的版本，例如不加權平均，我們還沒做。
[click] 第六，Fake-French 是造出來的語言：把法文的 token ID 平移到一段全新的 ID。它讓我們能控制詞彙重疊，但它不是一個真的、獨立演化的語言。
[click] 第七，均值偏移是一大塊，不是全部：核心實驗置中之後中文是 3.131，上界是 1.459，中間還有很大的距離。
[click] 最後，詞彙重疊的效果還是遠大於對齊：最好的核心實驗系列，Fake-French 仍然比共用子詞的真法文差約 2.1 bits/byte。幾何比想像中有用，但離「換一組列就等於原生」還很遠。
這也就是為什麼我們想試 B 和 C：如果列本身這麼重要，能不能用更少的代價、甚至不用 L_ban 語料，把列做得更好？接下來就是 Other Paths。
-->
