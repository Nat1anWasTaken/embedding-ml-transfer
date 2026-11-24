---
layout: default
---

<!-- PLAN（每一頁都要回答上一頁留下的問題）
實驗標記：每個實驗開頭一張「實驗：<名稱>」介紹頁（完整 ResultsExpTag），之後每張結果頁右上角放小標記；
比較多個實驗的頁，每一列都用同樣的實驗名稱標。藍色 chip = 與核心實驗不同的設定。
量法：每張圖標題下用 ResultsMeasure 寫清楚量的是哪一種——τ（放到 Floor→Upper 的尺上）、翻譯增益 Δ、
對齊部分（比打亂版好多少）、直接看 BPB。只有「比打亂版好多少」量的是對齊有沒有被用上，τ 本身不是。
（遮罩消融、純英文主體、置中的 τ 與 Δ 之坑等頁依使用者要求保留，待逐項決定。）
00 地圖：六個實驗各一列，藍色格子就是它跟核心實驗不同的那一項；底下一排量法圖例。
01 實驗 1 核心實驗：把列放回去；「我們預期它會失敗」製造張力。
02 核心 τ：明顯離開 Floor。
03 核心 Δ：理解端、生成端都顯著；判定等打亂對照補完。
04 實驗 2 全部可訓：embedding 一起訓，對齊還在嗎？
05 BLI 漂移：Fake-French BLI 5% 就歸零 → 對齊崩潰。
06 坑：拿 τ 量這個崩潰的模型 → 0.253 仍高於 Floor → τ 不量對齊。
07 τ 的底：Floor 的列全一樣，任何「彼此不同」的列都會贏 Floor。
08 打亂對照怎麼做。
09 比打亂版好多少：核心還在、全部可訓歸零。
10 回到核心實驗：打亂後 Δ 消失 → 核心實驗判「支持」。
11 實驗 3 只動 L_train 列：退一步。
12 BLI 還在，但比打亂版只好 0.042 → BLI ≠ 主體用得上對齊 → embedding 要凍住。
13 實驗 4 推論期置中：不重訓修語言均值偏移。
14 示意圖。
15 置中前後的 BPB（同一個核心 checkpoint）。
16 實驗 5 Qwen3 donor。
17 比自己的打亂版差多少：+0.547 ≈ 核心 +0.624 → 主體用上 Qwen3 的對齊（不放 τ 尺上：尺的兩端來自 Oracle 軌道）。
18 BPB：差距 0.77 → 置中後 0.11 → 是均值偏移。
19 實驗 6 解綁輸出頭。
20 三張卡。
21 理解 vs 生成。
22 收尾。
-->

# 八個實驗，都是*核心實驗改一項*

<div class="mt-6">
<ResultsExpTag map />
</div>

<div class="mt-8">
<ResultsMeasure legend :later="['align']" />
</div>

<!--
尺準備好了，現在進入結果。結果部分有八個實驗，先給大家一張地圖。每一列是一個實驗，欄位是它的設定：embedding 凍結還是可訓、embedding 從哪來、主體的訓練資料、訓練期 softmax 有沒有遮罩、輸出頭是不是綁定、模型規模、推論時有沒有做處理。

第一列是核心實驗，其他每一列都是「核心實驗改一項」，藍色的格子就是改的那一項。例如全部可訓是讓 embedding 一起訓練，Qwen3 donor 是把 embedding 換成 Qwen3 的。之後每張結果頁的右上角都會有這個標記，告訴大家現在看的是哪一個實驗的數字。

底下這排是我們會用到的量法，每張圖的標題下面都會寫它是哪一種：τ 是把結果放到 Floor 到 Upper 的尺上；翻譯增益 Δ 是給對的來源句、譯文好多少；直接看 BPB 就是同一批句子上的 bits per byte。灰掉的那一個，「比打亂版好多少」，會在實驗 2 之後加進來，到時候大家會看到為什麼需要它。
-->

---
layout: statement
kicker: 實驗 1 / 8
clicks: 1
---

# 實驗：核心實驗

主體從沒看過 L<sub>ban</sub>，把它的列放回去，能遷移嗎？

<ResultsExpTag exp="core" full />

<div v-click="1" class="mt-6 text-2xl text-[var(--google-muted)]">開跑前，我們預期它會失敗</div>

<!--
先看最核心的一組，我們叫它核心實驗，後面每個實驗都是拿它改一項來比：Oracle Embedding 完全凍結、主體只在 Multi-Train 的英西德俄上訓練 20B token、訓練期用 softmax 遮罩，主體一次都沒看過中文和 Fake-French 的 token。推論時，我們就把這些語言的列原封不動地放回去。

[click] 老實說，開跑前我們預期這個實驗會失敗。理由是 L_ban 的列因為語言均值偏移會聚成一團，主體只要學會避開那一個方向，就能把整個語言壓下去，原始解碼下 τ 應該接近 0。結果剛好相反，我們一個一個看。
-->

---
clicks: 2
---

<ResultsExpTag exp="core" />

# 語言內排序的 τ

<ResultsMeasure kind="tau" />

<div class="mx-auto" style="width: 46rem; view-transition-name: tau-line">
<RevealChart :rows="[{label:'真中文', value:0.297, ci:[0.291,0.302], color:'red'},{label:'Fake-French', value:0.436, ci:[0.431,0.442], color:'green'}]" :markers="[{value:0,label:'Floor',color:'gray'},{value:1,label:'Upper',color:'blue'}]" axis="τ（語言內排序）" :digits="3" :min="-0.2" :max="1" />
</div>

<div class="mt-2 text-lg text-[var(--google-muted)]">seed 42</div>

<!--
這就是 setup 那條 τ 數線：0 是 Floor，1 是看過 L_ban 的上界。

[互動] 剛才 setup 最後請大家下了注，在點下去之前再確認一次：一個從沒看過中文的主體，中文會落在這條線的哪裡？Fake-French 呢？

[click] 真中文 0.297，CI 是 0.291 到 0.302，下界過了 0.2。

[click] Fake-French 更高，0.436。換 seed 43 重跑，是 0.304 和 0.470，結論一樣。順帶一提，整體 BPB 算的 τ 是 0.307 和 0.328，但判定用的是語言內排序這一項。
-->

---
clicks: 4
---

<ResultsExpTag exp="core" />

# 翻譯增益 Δ − Δ<sub>floor</sub>

<ResultsMeasure kind="delta" note="減掉 Floor 自己的 Δ" />

<RevealChart :rows="[{label:'中文 → En', value:0.242, ci:[0.236,0.248], color:'red'},{label:'Fake → En', value:0.386, ci:[0.376,0.396], color:'green'},{label:'En → 中文', value:0.326, ci:[0.317,0.335], color:'red'},{label:'En → Fake', value:0.484, ci:[0.472,0.496], color:'green'}]" :markers="[{value:0,label:'Floor',color:'gray'}]" :min="0" :max="0.6" axis="Δ − Δ_floor（bits/byte）" :digits="3" />

<!--
τ 只看單語續寫。再來看翻譯增益 Δ：給真正的來源句，跟給一句長度相近但不相干的來源句，譯文的 BPB 差多少，再扣掉 Floor 自己的 Δ。

[click] 先看理解端，也就是 L_ban 翻成英文：輸出是英文，繞過了 tied 輸出端。中文 +0.242。[click] Fake-French +0.386，CI 都離 0 很遠。原始 Δ 大約是上界的六成，中文 0.270 對 0.447、Fake-French 0.388 對 0.645。

[互動] 那反過來，英文翻成 L_ban，需要模型自己吐出沒看過的 token，大家覺得會比較低還是比較高？

[click] 中文 +0.326，[click] Fake-French +0.484，反而比理解端高。En→Fake-French 的原始 Δ 是 0.491，甚至超過上界的 0.466。這跟我們「輸出端是瓶頸」的預期相反。
到這裡，兩個 seed 都一致：τ 明顯離開 Floor，翻譯增益也顯著大於 0。看起來很好，但先別急著下結論，核心實驗的最終判定我們等一下再回來補。先問一個實務上一定會被問的問題，也就是地圖上的第二個實驗。
-->

---
layout: statement
kicker: 實驗 2 / 8
---

# 實驗：全部可訓

embedding 也一起訓，對齊還在嗎？只改 E：可訓（softmax 跟著完整）

<ResultsExpTag exp="a4a" full />

<!--
到目前為止 embedding 都是凍結的。實務上大家一定會問：凍結太保守了吧？如果 embedding 也一起訓練，主體和 embedding 互相遷就，會不會更好？

所以第二個實驗是全部可訓：跟核心實驗比，只改 embedding 這一格：E 從 Oracle Embedding 起跑，全部可訓，embedding 的 weight decay 設 0，softmax 用完整的。這裡同時有兩股力：L_ban 列每一步被當負例推開，加上主體學會壓低它們，我們量的是兩者合計。（在實驗紀錄裡叫 A4a。）

要追蹤的是：訓練過程中，Oracle Embedding 原本的對齊還在不在。我們用 setup 關卡用過的 BLI 來量。
-->

---
clicks: 3
---

<ResultsExpTag exp="a4a" />

# 全部可訓：Fake-French 的 BLI

<ResultsMeasure kind="bli" />

<ResultsDriftChart :series="[{label:'全部可訓', color:'gray', points:[[0,0.800],[5,0.000],[10,0.000],[25,0.000],[50,0.000],[100,0.000]], labelDy:0},{label:'全部可訓 · LR ×0.1', color:'yellow', points:[[0,0.800],[5,0.057],[25,0.000],[50,0.000],[100,0.000]], labelDy:-64},{label:'全部可訓 · 中文', color:'red', points:[[0,0.484],[5,0.399],[10,0.304],[25,0.172],[50,0.063],[100,0.012]], dashed:true, labelDy:-30}]" y-label="BLI P@1" />

<!--
**BLI 是用來衡量 Embedding 內部不同語言彼此多對齊的指標**
**這裡是在看 Embedding 在被訓練完後破壞得怎麼樣**

我們在訓練中每 5% 存的 checkpoint 上追蹤 BLI。縱軸是 BLI P@1，起點是 Oracle Embedding 本身：Fake-French 0.800。

[click] 全部可訓、完整 softmax：訓練到 5%，Fake-French 的 BLI 就變成 0.000。我們就是在這時發現問題的。原因之一是 embedding 的 AdamW LR 0.006 是為隨機初始化校準的，對 RMS 0.095 的 Oracle Embedding，每步大約動 6%。

[互動] 那把 embedding LR 降 10 倍，救得回來嗎？

[click] 只是崩得慢一點：5% 時 0.057，25% 就歸零。所以主因是完整 softmax 每步把 L_ban 列當負例推開，LR 只決定崩多快；放大列範數也救不回來，是方向崩潰不是範數問題。原本計畫的「再加 weight decay」那一組也因此放棄，只是在已經崩潰的狀態上比較。

[click] 虛線是同一個 run 的中文 BLI，從 0.484 慢慢掉到 0.012。Fake-French 崩得快得多，我們的假說是 Fake-French 列本來就貼著真法文、西文的列，被推開的方向正好是遠離它的對齊夥伴：語言越近，在完整 softmax 下的對齊越脆弱。
所以答案很清楚：embedding 一起訓，對齊就沒了。那接下來的問題是：拿 setup 造好的那把尺，量這個對齊已經崩掉的模型，會量到什麼？
-->

---
clicks: 1
---

<ResultsExpTag :exp="['a4a', 'core']" />

# 同一把尺，量這個*崩潰*的模型

<ResultsMeasure kind="tau" />

<div class="mx-auto" style="width: 46rem; view-transition-name: tau-line">
<RevealChart :rows="[{label:'核心實驗 · 中文', value:0.297, color:'red', at:0},{label:'核心實驗 · Fake', value:0.436, color:'green', at:0},{label:'全部可訓 · 中文', value:0.253, ci:[0.249,0.257], color:'gray', at:1}]" :markers="[{value:0,label:'Floor',color:'gray'},{value:1,label:'Upper',color:'blue'}]" axis="τ（語言內排序）" :digits="3" :min="-0.2" :max="1" />
</div>

<!--
這裡有個坑，還是在全部可訓這個實驗裡。核心實驗的兩個點還留在線上，我把剛才那個全部可訓的模型也放上來。它的對齊是「完全崩掉」的：Fake-French 的 BLI 是 0，翻譯增益跟 Floor 一模一樣。

[互動] 一個完全沒用到對齊的模型，中文的 τ 應該落在哪？照理說應該貼著 Floor 吧？

[click] 0.253。還是明顯高於 Floor，跟核心實驗的 0.297 差不了多少。也就是說，如果只看 τ，一個對齊已經崩潰的模型看起來也有遷移。

[互動] 大家可以想一下，這把尺哪裡出了問題？

先講結論：τ 量的是「比 Floor 好多少」，不是「對齊有沒有被用上」，這兩件事不一樣。
-->

---
clicks: 2
---

<ResultsExpTag exp="a4a" />

# τ 的「底」從哪來

<ResultsRowsSketch />

<!--
問題出在 Floor 的定義。Floor 把所有 L_ban 的列都設成同一個平均向量，所以在語言內部，每個 token 的機率是一樣的，等於均勻分佈。

[click] 那只要這些列「彼此不同」，就算方向全是亂的，主體也能做點事，例如把前文出現過的 token 再複製一次。Floor 連這都做不到，所以任何「列彼此不同」的模型都會贏 Floor，τ 自動大於 0。

[click] 所以真正該比的零點不是 Floor，而是打亂對照：同一個 checkpoint，把 L_ban 的列在組內隨機互換，向量本身都還在，只是 token 對到錯的列。從打亂走到原始，這一段才是「對齊」貢獻的。
-->

---
clicks: 2
---

<ResultsExpTag exp="a4a" />

# 打亂對照：*τ<sub>shuffle</sub>* 怎麼來

<div class="text-lg text-[var(--google-muted)] -mt-1 mb-2">只拿掉「哪個 token 對到哪一列」，其他全部保留</div>

<ResultsShuffleProcedure />

<!--
這兩頁是全部可訓這個實驗逼出來的量尺修正，之後每個實驗都會用到。所以我們另外做了一個 tau_shuffle。

因為我們想問的是「對齊有沒有被用上」，而 Floor 拿掉的東西太多了：它把 L_ban 的列全換成同一個向量，同時拿掉了「對齊」和「列彼此不同」兩件事。任何模型贏 Floor，都分不出是贏在哪一件。我們需要一個對照組，只拿掉對齊、其他什麼都不動。

先看左邊。主體從頭到尾看到的只有向量。藍點是它訓練時看過的 L_train 列；紅點是 L_ban 的列，主體一次都沒看過，但因為 embedding 是預先對齊的，「博物館」的向量就落在「museum」旁邊。

[click] 打亂就是這樣：向量一個都不動，只把紅色的標籤換到別的紅點上。「博物館」現在拿到的是原本「山」的向量。具體做法在右邊：拿同一個訓練好的 checkpoint，不重訓，把 L_ban 的列在組內隨機重排，中文那 28,677 列彼此互換、Fake-French 那 29,622 列彼此互換，每一列都換到別的位置。因為是 tied，輸入端和輸出端一起換。L_train 的列完全不動，我們也驗證過英、西、德、俄、法的 BPB 和打亂前逐位元相同。然後用跟 τ 一樣的方法，在同一批 FLORES+ 句子上算 BPB；打亂是隨機的，所以用 seed 0 到 4 各打亂一次取平均。

[互動] 這時候可能會有人覺得：mapping 都改了，當然會變差，這有證明什麼嗎？

這不是理所當然的。關鍵是：這些紅點，主體一次都沒看過。零洩漏加上訓練期遮罩，中文和 Fake-French 的列從來沒進過主體。如果主體只是把它們當成一批陌生輸入，那「博物館」拿到哪一個陌生向量都一樣陌生，怎麼排都不該有差別，打亂前後的 BPB 應該一樣。如果打亂會變差，唯一的理由是正確的向量「位置有意義」：「博物館」的向量落在主體學過的「museum」附近，主體就把它當 museum 用；打亂後它拿到落在「mountain」旁邊的向量，就被理解成別的東西。

[click] 結果：核心實驗打亂後，語言內 BPB 中文變差 0.624、Fake-French 變差 1.482，CI 都很窄。而剛才那個對齊已經崩潰的全部可訓模型，做一模一樣的打亂，中文只變了 +0.0001，CI 含 0，Fake-French 也只有 +0.0018。mapping 一樣被改了，結果幾乎不變，因為它的紅點早就不在對的位置，換來換去都一樣沒意義。同樣的操作，一個沒差、一個差很多，這才是打亂能證明東西的地方：它量的是主體有沒有透過和看過的列之間的相對位置，去解讀沒看過的列。
-->

---
clicks: 2
---

<ResultsExpTag :exp="['a4a', 'core']" />

# 比打亂版好多少：*對齊部分*

<ResultsMeasure legend />

<div class="text-base text-[var(--google-muted)] mt-2 mb-1">

$\tau - \tau_{\text{shuffle}} = (\text{BPB}_{\text{shuffle}} - \text{BPB}_x)\,/\,(\text{BPB}_{\text{floor}} - \text{BPB}_{\text{upper}})$ · 打亂取 5 個 seed 平均

</div>

<div class="mx-auto" style="width: 46rem; view-transition-name: tau-line">
<RevealChart :rows="[{label:'核心實驗 · 中文', value:0.203, ci:[0.198,0.208], color:'red', at:1},{label:'核心實驗 · Fake', value:0.405, ci:[0.400,0.410], color:'green', at:1},{label:'全部可訓 · 中文', value:0.000, color:'gray', at:2},{label:'全部可訓 · Fake', value:0.000, color:'gray', at:2}]" :markers="[{value:0,label:'打亂',color:'gray'},{value:1,label:'Upper',color:'blue'}]" axis="對齊部分（語言內排序；0 = 跟打亂一樣）" :digits="3" :min="-0.2" :max="1" />
</div>

<!--
量法的最後一種補上了：比打亂版好多少。把打亂前後的 BPB 差，除以同一個 Floor 到 Upper 的距離，就是畫面上的 τ − τ_shuffle，我們叫它「對齊部分」。數線的零點從 Floor 換成了打亂：0 代表跟打亂一樣、對齊完全沒被用上。從現在起，要問「對齊有沒有被用上」，只看這一把尺；τ 本身只告訴我們比 Floor 好多少。

[click] 核心實驗扣掉之後還剩多少？中文從 0.297 降到 0.203，Fake-French 從 0.436 降到 0.405。中文縮水比較多，Fake-French 幾乎沒掉。

[click] 全部可訓這個崩潰的模型，兩個條件都是 0.000，它的 0.253 全部是底。所以核心結論還在，只是中文對齊本身的貢獻大約 0.20 到 0.22，沒有原始 τ 看起來那麼大。另外注意，Fake-French 的對齊部分大約是中文的兩倍，這在兩個 seed 和其他 run 上都一致：語言越近，語言內排序遷移越多。
-->

---
clicks: 2
---

<ResultsExpTag exp="core" />

# 打亂之後，翻譯增益消失

<ResultsMeasure kind="delta" />

<RevealChart :rows="[{label:'中文→En 原始', value:0.270, color:'red', at:1},{label:'中文→En 打亂', value:0.029, color:'gray', at:2},{label:'Fake→En 原始', value:0.388, color:'green', at:1},{label:'Fake→En 打亂', value:0.003, color:'gray', at:2}]" :markers="[{value:0,label:'0',color:'gray'}]" :min="0" :max="0.5" axis="Δ（bits/byte）" :digits="3" />

<!--
有了打亂對照，我們回到實驗 1，核心實驗，把剛才留著的判定補完：用打亂對照檢查翻譯增益。

[click] 這是核心實驗原始的 Δ：中文翻英文 0.270，Fake-French 翻英文 0.388。

[click] 打亂之後，中文掉到 0.029，Fake-French 掉到 0.003，幾乎歸零，跟 Floor 差不多。所以我們可以比較有信心地說：遷移是來自 token 對到「對的那一列」，也就是幾何對齊，而不只是列彼此不同。
高於 Floor、翻譯增益顯著、又贏過打亂對照，兩個 seed 也一致，所以核心實驗我們判定「支持假設」。
那回到可訓 embedding 的問題：全部可訓會崩，那退一步呢？這就是第三個實驗。
-->

---
layout: statement
kicker: 實驗 3 / 8
---

# 實驗：只動 L<sub>train</sub> 列

全部可訓會崩；退一步，L<sub>ban</sub> 列凍住、只讓 L<sub>train</sub> 列動呢？

<ResultsExpTag exp="a4c" full />

<!--
第三個實驗是退一步的版本：跟核心實驗比，只改 embedding 這一格，從「全部凍結」變成「只有 L_train 的列可以動」，L_ban 的列保持凍結，訓練期照樣用遮罩，embedding weight decay 是 0。在遮罩下 L_ban 列本來就收不到梯度，所以它就是「全部可訓」換成遮罩的樣子（實驗紀錄叫 A4c）。直覺上，L_ban 列沒被推開，對齊應該保得住。
-->

---
clicks: 2
---

<ResultsExpTag exp="a4c" />

# BLI 還在，*遷移*呢？

<div class="grid grid-cols-2 gap-8 mt-8 text-center">
  <div>
    <ResultsMeasure kind="bli" />
    <div class="text-xl text-[var(--google-muted)] mt-3">Fake-French</div>
    <div class="text-7xl font-bold text-[var(--google-green)] mt-2">0.782</div>
    <div class="text-lg text-[var(--google-muted)] mt-2">起點 0.800</div>
  </div>
  <div v-click="1">
    <ResultsMeasure kind="align" />
    <div class="text-xl text-[var(--google-muted)] mt-3">Fake-French</div>
    <div class="text-7xl font-bold text-[var(--google-red)] mt-2">0.042</div>
    <div class="relative mx-auto mt-4 w-[17rem] h-10">
      <div class="absolute left-0 right-0 top-4 h-1.5 rounded-full bg-[var(--google-surface)]"></div>
      <div class="absolute left-0 top-4 h-1.5 rounded-full bg-[var(--google-green)]" style="width: 40.5%"></div>
      <div class="absolute left-0 top-4 h-1.5 rounded-full bg-[var(--google-red)]" style="width: 4.2%"></div>
      <div class="absolute top-0 text-[13px] text-[var(--google-green)] -translate-x-1/2" style="left: 40.5%">核心實驗 0.405</div>
      <div class="absolute top-7 left-0 text-[12px] text-[var(--google-muted)] -translate-x-1/2">0 = 跟打亂一樣</div>
      <div class="absolute top-7 right-0 text-[12px] text-[var(--google-muted)] translate-x-1/2">1 = Upper</div>
    </div>
  </div>
</div>

<div v-click="2" class="mt-10 text-center text-2xl font-semibold">BLI 保持 ≠ 主體還能用上對齊</div>

<!--
還是只動 L_train 列這個實驗。先看 BLI：最近鄰關係幾乎完全保住，Fake-French 0.800 到 0.782，中文 0.484 到 0.462。我們開跑前也預期差距不大，因為 Oracle Embedding 本來就是在同一批 L_train 資料上訓練的。

[互動] 所以對齊保住了嗎？遷移應該跟核心實驗差不多吧？

[click] 右邊用的是前兩頁那把尺：同一個 checkpoint 跟它自己的打亂版比，BPB 好了多少，再除以 Floor 到 Upper 的距離。0 代表跟打亂一樣、對齊完全沒被用上；核心實驗的 Fake-French 是 0.405。這個 run 只剩 0.042，中文 0.018（核心實驗 0.203），幾乎貼著 0，第二個 seed 一樣。翻譯增益 Δ − Δ_floor 也只剩 +0.006。

[click] 所以 BLI 看起來沒事，不代表主體還用得上對齊。BLI 只看 embedding 內部的最近鄰關係；L_train 的列一動，L_ban 的列原本對齊的對象就跑掉了，逐列 cosine 其實已經掉到 0.54。
-->

---
clicks: 2
---

<ResultsExpTag exp="a4c" />

# 這裡有個坑：L<sub>train</sub> 列 RMS ×5.6

<div class="text-xl text-[var(--google-muted)] mt-1">L<sub>ban</sub> 列凍在原尺度 → 輸入端相對變得很小</div>

| 語言內 BPB 配對差 | 真中文 | Fake-French |
| --- | --- | --- |
| 只動 L<sub>train</sub> 列 − 核心實驗 | +0.116 | +0.884 |
| <span v-click="1">放大輸入 − 只動 L<sub>train</sub> 列</span> | <span v-click="1">−0.335</span> | <span v-click="1">−0.266</span> |
| <span v-click="2">放大輸入 − 核心實驗</span> | <span v-click="2" class="font-bold text-[var(--google-green-dark)]">−0.219</span> | <span v-click="2" class="font-bold text-[var(--google-red)]">+0.618</span> |

<!--
那為什麼 BLI 不變、cosine 卻掉這麼多？[互動] 大家可以想一下 CSLS 這個指標對什麼不敏感。

答案是尺度。embedding weight decay 是 0，L_train 列的 RMS 長了大約 5.6 倍，凍結的 L_ban 列停在原本的尺度，所以在輸入端 L_ban 的 token 相對變得很小。「只動 L_train 列」減核心實驗，其實混了「範數不配」和「方向漂移」兩件事。

[click] 我們做了一個零訓練成本的尺度對照：只在輸入端把 L_ban 列乘一個純量，讓它的 RMS 跟 L_train 列一樣。中文好了 0.335、Fake-French 好了 0.266。

[click] 跟核心實驗比：中文反而好 0.219，Fake-French 還是差 0.618。所以中文的損失全來自範數不配；Fake-French 大約三成是尺度、七成留下來。中文的這 0.219 是對齊回來了，還是範數補回之後列又可分、也就是 τ 的底，我們還沒拆，需要對放大後的模型再跑打亂對照。要乾淨分開，得重訓一個範數對齊的版本，大約 40 GPU-hr，還沒決定。
可訓 embedding 這條線的結論是：要用對齊，embedding 就得凍住。所以接下來回到凍結的設定，看幾個細節，先是第四個實驗。
-->

---
layout: statement
kicker: 實驗 4 / 8
---

# 實驗：遮罩消融

E 凍結時，完整 softmax 的壓制有多大？只改：拿掉訓練期遮罩（150M）

<ResultsExpTag exp="mask" full />

<!--
剛才看到，可訓的 embedding 在完整 softmax 下會被推開。那 embedding 凍結的時候呢？Setup 提過，就算列不能動，L_ban 列每一步還是負例，主體會學到把 hidden state 推離它們，所以我們才加了訓練期遮罩。

第四個實驗就是量這個遮罩到底幫了多少。這是一個 150M、5B token 的小規模消融，兩條 run 只差「softmax 有沒有遮罩」，embedding 都凍結。注意規模那一格也是藍的：這組的數字只能跟彼此比，不能直接跟 300M 的核心實驗比。
-->

---
clicks: 3
---

<ResultsExpTag exp="mask" />

# 訓練期 softmax 遮罩幫了多少

<RevealChart :rows="[{label:'真中文', value:-0.2858, ci:[-0.2998,-0.2722], color:'red'},{label:'Fake-French', value:-0.2252, ci:[-0.2336,-0.2166], color:'green'},{label:'L_train 7 項', value:0.0011, ci:[0.0004,0.0018], color:'gray'}]" :markers="[{value:0,label:'無差異',color:'gray'}]" :min="-0.35" :max="0.05" axis="遮罩 − 完整 softmax（BPB，越負越好）" :digits="3" />

<div class="text-lg text-[var(--google-muted)]">5B tokens · 兩條 run 只差遮罩開關</div>

<!--
遮罩消融的結果。縱軸是遮罩版減完整 softmax 版。中文與 Fake-French 看的是語言內排序 BPB，L_train 看的是整體 BPB。

[互動] 大家覺得遮罩會改善多少？會不會只是改善「語言選擇」那一項？

[click] 中文語言內排序好了 0.286。[click] Fake-French 好了 0.225。[click] L_train 只差 0.0011，遠低於我們事先容許的 0.005。

有一點不符合預期：改善幾乎全在語言內排序，語言選擇中文是 +0.000、Fake-French 只有 −0.031。代價是 En→Fake-French 的 Δ 少了 0.024、真法文 BPB 多了 0.011。這是單一 seed、150M，所以只當方向。
不過這個消融裡還藏了一個更怪的結果。
-->

---
layout: default
clicks: 2
---

<ResultsExpTag exp="mask" />

<div class="text-lg text-[var(--google-muted)]">完整 softmax 那條 run</div>

# 把中文列<br>全換成*平均值*

<div v-click="1" class="mt-4 text-3xl font-semibold text-[var(--google-red)]">語言內 BPB 反而好 0.085</div>
<div v-click="2" class="mt-2 text-2xl text-[var(--google-muted)]">遮罩版：差 0.077（正常方向）</div>

<!--
還是遮罩消融，在完整 softmax 那條 run 上，我們做「自身 Floor」：用跟 Floor 一樣的作法，把它自己的中文列全部換成平均值，也就是把中文的資訊整個拿掉。

[互動] 拿掉資訊之後，中文應該變差對吧？大家覺得會差多少？

[click] 結果反而變好了 0.085 bits/byte。意思是：完整 softmax 的壓制不只壓「語言選擇」，還把中文內部的排序弄亂了，亂到比「完全沒有中文資訊」還糟。

[click] 遮罩版就沒有這個現象：原始模型比自身 Floor 好 0.077，是正常的方向。這也是為什麼主線全部改用遮罩。
遮罩的事講完，接下來是第五個實驗，換另一個變因：主體的訓練資料。多語主體有沒有幫助？
-->

---
layout: statement
kicker: 實驗 5 / 8
---

# 實驗：純英文主體

主體訓練時看過多種外語，會比較會用對齊嗎？只換：訓練資料改成 20B 純英文

<ResultsExpTag exp="mono" full />

<!--
第五個實驗換另一個變因：主體的訓練資料。這就是 setup 裡那份 Mono-Train 的用途：只改一件事，把 Multi-Train 換成同樣 20B 的純英文，其他跟核心實驗一模一樣。地圖上變藍的只有「訓練資料」這一格。
-->

---
clicks: 3
---

<ResultsExpTag :exp="['mono', 'core']" />

<div style="view-transition-name: a2-title">

# 純英文主體

</div>

<RevealChart :rows="[{label:'真中文', value:-0.230, ci:[-0.242,-0.218], color:'red'},{label:'Fake-French', value:-0.221, ci:[-0.234,-0.207], color:'green'},{label:'真法文', value:0.059, ci:[0.054,0.065], color:'yellow'}]" :markers="[{value:0,label:'與多語主體相同',color:'gray'}]" :min="-0.3" :max="0.1" axis="純英文 − 多語（語言內排序 BPB，負 = 純英文較好）" :digits="3" />

<!--
純英文主體跟核心實驗直接比語言內排序 BPB。

[互動] 大家覺得多語主體會贏多少？我們開跑前的預期是純英文不會比較好，差距在真中文最大。

[click] 結果真中文反而是純英文好 0.230。[click] Fake-French 也是純英文好 0.221。[click] 只有共用 token 的真法文符合預期，多語主體好 0.059。

依照事先的判準字面，這是否定假設。不過這裡要提醒 seed 噪音：純英文主體只有一個 seed，拿它跟核心實驗的第二個 seed 比，Fake-French 只剩 −0.098，落在兩個 seed 之間 0.123 的噪音裡。bootstrap CI 只含句子抽樣，不含訓練隨機性，所以單 seed 的差距要大過這個量級才算數。
那純英文主體到底贏在哪？
-->

---
clicks: 2
---

<ResultsExpTag :exp="['mono', 'core']" />

<div style="view-transition-name: a2-title">

# 純英文主體：對齊部分

</div>

<div class="mx-auto" style="width: 46rem; view-transition-name: tau-line">
<RevealChart :rows="[{label:'多語 · 中文', value:0.203, color:'red', at:1},{label:'純英文 · 中文', value:0.201, ci:[0.197,0.205], color:'red', at:2},{label:'多語 · Fake', value:0.405, color:'green', at:1},{label:'純英文 · Fake', value:0.407, ci:[0.404,0.411], color:'green', at:2}]" :markers="[{value:0,label:'打亂',color:'gray'},{value:1,label:'Upper',color:'blue'}]" axis="τ − τ_shuffle（語言內排序）" :digits="3" :min="-0.2" :max="1" />
</div>

<!--
我們把剛才學到的工具拿出來，只看對齊部分。

[click] 多語主體：0.203 和 0.405。

[click] 純英文主體：0.201 和 0.407，幾乎完全重疊。打亂造成的損失也一樣，中文 +0.617 對 +0.624。所以多語主體沒有讓主體更會利用對齊。純英文主體的原始 τ 比較高，是 0.371 和 0.496，多出來的全在底：中文的 τ_shuffle 是 0.171，核心實驗只有 0.094。


候選解釋是吃了 20B 英文、通用 LM 比較強，或者「在上下文中複製」做得比較好。多語主體真正贏的是翻譯增益：理解端 Δ 中文多 0.052、Fake-French 多 0.194。
-->

---
clicks: 4
---

<ResultsExpTag :exp="['mono', 'core']" />

# Fake-French：排得對，卻挑錯語言

<ResultsStackedBars :rows="[{label:'Floor', parts:[{value:4.657,color:'green'},{value:1.024,color:'red'}], total:5.681},{label:'多語主體', parts:[{value:3.061,color:'green'},{value:1.090,color:'red'}], total:4.150},{label:'純英文主體', parts:[{value:2.840,color:'green'},{value:0.666,color:'red'}], total:3.506},{label:'Upper', parts:[{value:0.997,color:'green'},{value:0.019,color:'red'}], total:1.016}]" :legend="[{label:'語言內排序',color:'green'},{label:'語言選擇',color:'red'}]" :max="6" />

<!--
純英文主體另一個優勢在 setup 講過的「語言選擇」。這張是 Fake-French 的 BPB，拆成語言內排序（綠）和語言選擇（紅）兩段。

[click] Floor：語言內 4.657、選擇 1.024，總共 5.681。

[click] 多語主體：語言內大幅進步到 3.061，可是語言選擇是 1.090，比 Floor 還差。也就是說它在 Fake-French 那組 token 裡排得不錯，但機率常常放到熟悉的拉丁字母 token 上，挑錯語言。這一項大約占 Fake-French 整體 BPB 的四分之一。中文的語言選擇只有 0.013，因為漢字是一整群獨立的列，很容易整群挑中。兩股力一抵銷，整體 τ 中文和 Fake-French 幾乎一樣，0.307 對 0.328。

[click] 純英文主體的語言選擇只有 0.666。我們的假說是多語主體學過西德文，遇到法文式的上下文就把機率分給西德和真法文的 token，純英文主體的競爭者比較少；這個還沒驗證。

[click] 上界的選擇項只有 0.019，所以這是「主體沒看過 Fake-French」造成的，不是條件本身的限制。

（註：Floor 的語言內 BPB，B0-Lower 頁寫 4.657、跨條件表寫 4.656，這裡用 4.657。）
-->

---
layout: fact
clicks: 1
---

<ResultsExpTag exp="core" />

# +2.108

bits/byte：Fake-French 仍比真法文差

<div v-click="1" class="mt-3 text-xl text-[var(--google-muted)]">Floor 差 3.630 → 只追回 42%</div>

<!--
Fake-French 和真法文是同一批句子、同一個分母，所以可以直接相減，這是詞彙重疊效果最乾淨的量法。

核心實驗的 Fake-French，還是比真法文差 2.108 bits/byte，CI 是 2.078 到 2.138。

[click] Floor 的差距是 3.630，所以對齊 embedding 只追回 42%，第二個 seed 是 46%，純英文主體 61%。反過來看，看過 Fake-French 的上界，Fake-French 反而比真法文好 0.898。真法文幾乎不受 embedding 怎麼處理影響，所有軌道 1 的 run 都在 2.02 到 2.15 之間：共用子詞已經把大部分能力帶過去了。結論是：詞彙重疊本身的效果，遠大於對齊 embedding 帶來的遷移。
回頭看剛才兩個缺口：Fake-French 會挑錯語言、而且 L_ban 的列整群聚在一起，跟 L_train 的列不在同一個位置。這正是 Intro 假設的後半句：要把語言子空間的偏移拿掉，生成才會出來。那能不能不重訓，直接在推論時修？
-->

---
layout: statement
kicker: 實驗 6 / 8
---

# 實驗：推論期置中

不重訓，推論時把 L<sub>ban</sub> 的列整群平移，能補回多少？

<ResultsExpTag exp="center" full />

<!--
Related Works 講過語言均值偏移：每個語言的列在空間裡有自己的平均位置，L_ban 的列整群偏在一邊，跟主體熟悉的 L_train 列不在同一個位置。這正是 Intro 假設的後半句。

所以第六個實驗不重訓任何東西：拿同一個核心實驗的 checkpoint，只在推論時多做一步置中。地圖上唯一變藍的就是「推論期」這一格。
-->

---
clicks: 1
---

<ResultsExpTag exp="center" />

# 推論期 Mean-Centering

<div style="view-transition-name: mean-shift">
<ResultsMeanShift :shift-at="1" />
</div>

<!--
這張是示意圖，不是實際資料。左邊是原本的樣子：L_ban 的列聚成一群，平均位置跟 L_train 的不一樣。

[click] Mean-Centering 就是推論時，把 L_ban 自己的列整群平移 μ(L_train) − μ(L_ban)，群內的相對結構完全不動。因為是 tied，輸入輸出一起移。均值用 token 頻率加權。要誠實講一點：μ(L_ban) 用的是 Oracle-Train 裡中文和 Fake-French 的 token 計數，跟評估集分開，但不是嚴格的「零目標語料」。整個實驗不需要重訓，加起來 0.13 GPU-hr。
-->

---
clicks: 3
---

<ResultsExpTag exp="center" />

<div class="flex items-start justify-between">

# 核心實驗的 2×2

<div style="view-transition-name: mean-shift" class="w-48 -mt-2">
<ResultsMeanShift :shift-at="0" bare />
</div>

</div>

| BPB | 原始 | 限定語言 | 置中 | 置中 + 限定語言 |
| --- | --- | --- | --- | --- |
| 真中文 | 3.632 | <span v-click="1">3.619</span> | <span v-click="2">3.131</span> | <span v-click="3" class="font-bold text-[var(--google-blue)]">3.061</span> |
| Fake-French | 4.150 | <span v-click="1">3.061</span> | <span v-click="2">3.306</span> | <span v-click="3" class="font-bold text-[var(--google-blue)]">2.626</span> |

<!--
同一個核心實驗的 checkpoint，權重不動，只改解碼。兩個開關：第一個是「限定語言」，推論時把機率重新歸一化到該語言那組 token 上，等於直接告訴模型「答案一定是這個語言」。注意它和 setup 的訓練期遮罩不是同一件事，那個是訓練時用的。第二個開關是置中。

[click] 限定語言：中文幾乎沒變，3.632 到 3.619，因為中文的語言選擇本來就只有 0.013；Fake-French 從 4.150 降到 3.061，就是把剛才那段「挑錯語言」直接拿掉。

[click] 置中：中文降到 3.131，Fake-French 3.306。中文的進步幾乎全在語言內排序（−0.557），Fake-French 兩項各半。

[click] 兩個都做最好：3.061 和 2.626。代價是 L_train 多 0.009、真法文多 0.071，因為平移後的 L_ban 列會搭走共用 token 的機率。
-->

---
clicks: 2
---

<ResultsExpTag exp="center" />

# 置中前後的 τ

<div class="mx-auto" style="width: 46rem; view-transition-name: tau-line">
<RevealChart :rows="[{label:'真中文 原始', value:0.307, ci:[0.302,0.312], color:'gray', at:1},{label:'真中文 置中', value:0.467, color:'red', at:2},{label:'Fake 原始', value:0.328, ci:[0.324,0.332], color:'gray', at:1},{label:'Fake 置中', value:0.509, color:'green', at:2}]" :markers="[{value:0,label:'Floor',color:'gray'},{value:1,label:'Upper',color:'blue'}]" axis="τ（整體 BPB）" :digits="3" :min="-0.2" :max="1" />
</div>

<!--
注意這張的 τ 是用整體 BPB 算的，不是前面判定用的語言內排序，所以起點是 0.307 和 0.328。Floor 和上界本身沒有置中。

[click] 原始：中文 0.307、Fake-French 0.328。

[click] 置中後：中文 0.467、Fake-French 0.509，都往上界方向走了一大段。

但置中後離上界還很遠：中文 3.131 對上界 1.459，Fake-French 3.306 對 1.016。語言均值偏移是一大塊，但不是全部。另外也不是每個模型都受益：純英文主體的 Fake-French 置中後反而變差 0.174，假說是它本來就比較會挑中 Fake-French 那組 token，再往同方向推就過頭了。
-->

---
clicks: 2
---

<ResultsExpTag exp="center" />

# 這裡有個坑：En → L 的 Δ 變小了

| En → L | <span v-click="2">給平行英文</span> | <span v-click="2">給無關英文</span> | Δ |
| --- | --- | --- | --- |
| 中文 | <span v-click="2">3.280 → 2.725</span> | <span v-click="2">3.647 → 3.078</span> | <span v-click="1">0.367 → 0.352</span> |
| Fake-French | <span v-click="2">3.515 → 2.945</span> | <span v-click="2">4.006 → 3.291</span> | <span v-click="1" class="text-[var(--google-red)] font-semibold">0.491 → 0.345</span> |

<div v-click="2" class="mt-6 text-xl text-[var(--google-muted)]">兩側都大幅變好，只是「無關」那側進步更多</div>

<!--
推論期置中的事先判準有兩個條件：BPB 要變好，而且 En→L_ban 的翻譯增益 Δ 要變大。

[click] BPB 確實變好了，但 Δ 變小了：中文 0.367 到 0.352，Fake-French 0.491 到 0.345，CI 都不含 0。照字面這是「不確定」。

[互動] 置中真的讓翻譯變差了嗎？還是這個指標本身有問題？Δ 是兩個 BPB 的差，大家可以想想兩邊各自發生什麼事。

[click] 拆開看兩側的絕對 BPB：給平行英文時，譯文 BPB 大幅變好，中文少了 0.554、Fake-French 少了 0.570；只是給無關英文那側進步更多，所以差距縮小。我們的假說是：沒置中時，平行英文除了提供內容，也順便幫模型「找到正確的語言區域」；置中直接把這件事做掉了，平行句的額外幫助就變小。所以最後判定支持，之後的規則是用 Δ 判斷介入效果時，一定要同時報兩側的絕對 BPB。
-->

---
layout: statement
kicker: 實驗 7 / 8
---

# 實驗：Qwen3 donor

現實中沒有 Oracle：凍結 Qwen3-0.6B-Base 的 embedding，從頭訓練我們自己的主體

<ResultsExpTag exp="qwen" full />

<!--
到這裡為止用的都是 Oracle Embedding。但 setup 提過，它是一個理想化的 donor：來自看過 L_ban 的上界模型，現實中沒有這種東西。所以第七個實驗問：直接拿開源的 Qwen3-0.6B-Base 的 embedding，凍結，然後用同樣的架構、同樣的 Multi-Train 從頭訓練一個我們自己的 24 層主體，行不行？這個訓練本身就是實驗，地圖上變藍的只有「E 來源」這一格。

它可能更好：36T token、119 種語言，幾何品質理論上比我們自訓的好。也可能更差：它原本的架構跟我們的不一樣，列的尺度也不同。

要注意這一格其實是「整包換掉」：預訓練資料、語言數、原架構、列尺度，而且 Qwen 預訓練時本來就看過中文。詞表也沒有 Fake-French 的列，所以這個實驗只看真中文。
-->

---
clicks: 1
---

<ResultsExpTag :exp="['qwen', 'core']" />

# Qwen3 donor：主體也用上了它的對齊嗎？

<ResultsMeasure kind="shuffle" />

<RevealChart :rows="[{label:'核心實驗', value:0.624, ci:[0.609,0.638], color:'red', at:0},{label:'Qwen3 donor', value:0.547, ci:[0.524,0.570], color:'yellow', at:1}]" :markers="[{value:0,label:'跟打亂一樣',color:'gray'}]" :min="-0.1" :max="0.8" axis="打亂 − 原始 · 真中文語言內 BPB（越大 = 越用得上對齊）" :digits="3" />

<!--
Qwen3 donor 這個 run 沒有自己的上界和 Floor，所以我們不把它放到 τ 那把尺上：那把尺的兩端都是用 Oracle 軌道做的，對它不公平。我們只做這個 run 自己內部就能比的事：跟它自己的打亂版比。

核心實驗打亂之後，中文語言內 BPB 變差 0.624。

[互動] Qwen3 donor 打亂之後會差多少？如果主體沒用上 Qwen3 的幾何，應該接近 0。

[click] +0.547，CI 很窄，跟核心實驗同一個量級。所以主體也用上了 Qwen3 embedding 的對齊。翻譯增益也一樣：中文翻英文的 Δ − Δ_floor 是 +0.209，核心實驗是 +0.242。

那幾何可用，它的 BPB 表現如何？
-->

---
clicks: 2
---

<ResultsExpTag :exp="['qwen', 'core']" />

# Qwen3 的中文列差在哪

<ResultsMeasure kind="bpb" />

<RevealChart :rows="[{label:'核心實驗 原始', value:3.632, color:'red', at:1},{label:'Qwen3 donor 原始', value:4.402, color:'yellow', at:1},{label:'核心實驗 置中', value:3.131, color:'red', at:2},{label:'Qwen3 donor 置中', value:3.239, color:'yellow', at:2}]" :markers="[]" :min="2.5" :max="5" axis="真中文 BPB（越低越好）" :digits="3" :label-width="210" />

<!--
這裡直接看 BPB，同一批句子，越低越好。

[click] 原始中文 BPB：核心實驗 3.632，Qwen3 donor 4.402，差 0.77，看起來差很多。

[互動] 大家覺得這 0.77 是對齊品質比較差，還是別的原因？

[click] 把剛才實驗 6 的推論期置中套上去，兩個都置中之後：核心實驗 3.131、Qwen3 donor 3.239，只差大約 0.11。所以原始差距 0.77 幾乎全部是 Qwen3 中文列的語言均值偏移，不是對齊品質。結論是開源 donor 不能「直接借用」，但幾何是可用的，要配合置中。

最後一個問題，回到 tie 本身。
-->

---
layout: statement
kicker: 實驗 8 / 8
---

# 實驗：解綁輸出頭

Tie 是遷移的必要條件嗎？只改輸出端：不再直接用凍結的 E

<ResultsExpTag exp="untie" full />

<!--
最後一個實驗：tie 是零樣本遷移的必要條件嗎？先釐清一件事：輸入和輸出都凍結的時候，有沒有 tie 根本沒有差別。所以這個消融真正在改的，是「輸出端用的是不是那個對齊好的矩陣」。地圖上變藍的只有「輸出頭」這一格，輸入端還是凍結的 Oracle Embedding。
-->

---
clicks: 2
---

<ResultsExpTag exp="untie" />

# 把 tie 拆開：其實在改輸出端

<div class="grid grid-cols-3 gap-5 mt-8 text-center">
  <div class="rounded-2xl p-5 bg-[var(--google-blue-soft)] border border-[var(--google-blue)]">
    <div class="text-2xl font-bold">核心實驗</div>
    <div class="mt-4 text-lg">輸入　E（凍結）</div>
    <div class="text-lg">輸出　E（凍結）</div>
  </div>
  <div v-click="1" class="rounded-2xl p-5 border-2 border-[var(--google-red)]">
    <div class="text-2xl font-bold">隨機輸出頭</div>
    <div class="mt-4 text-lg">輸入　E（凍結）</div>
    <div class="text-lg">輸出　獨立 head</div>
  </div>
  <div v-click="2" class="rounded-2xl p-5 border-2 border-[var(--google-yellow)]">
    <div class="text-2xl font-bold">加線性層</div>
    <div class="mt-4 text-lg">輸入　E（凍結）</div>
    <div class="text-lg">輸出　E(I + Δ)</div>
  </div>
</div>

<!--
解綁輸出頭有兩個版本，跟綁定的核心實驗並排。

[click] 第一個是隨機輸出頭：輸出頭獨立隨機初始化再訓練。因為沒有 L_ban 資料，輸出頭裡 L_ban 的列會停在隨機初始值。

[click] 第二個是加線性層：輸出仍用凍結的 E，但前面插一層可訓的 d×d 線性層，初始化成 identity，寫成 I + Δ，Δ 從 0 開始、用 Muon 訓練。靈感是中間層在概念空間，看這一層能不能補上「轉回目標語言的最後一哩」。
-->

---
clicks: 4
---

<ResultsExpTag :exp="['untie', 'core']" />

# 理解 vs 生成 · Fake-French

<ResultsMeasure kind="delta" note="減掉 Floor 自己的 Δ" />

<div class="w-[92%] mx-auto">
<RevealChart :rows="[{label:'理解 · 核心實驗', value:0.386, color:'blue', at:1},{label:'理解 · 隨機輸出頭', value:0.447, ci:[0.436,0.457], color:'gray', at:2},{label:'理解 · 加線性層', value:0.414, ci:[0.404,0.424], color:'yellow', at:2},{label:'生成 · 核心實驗', value:0.484, color:'blue', at:3},{label:'生成 · 隨機輸出頭', value:-0.181, ci:[-0.187,-0.174], color:'gray', at:4},{label:'生成 · 加線性層', value:0.357, ci:[0.346,0.368], color:'yellow', at:4}]" :markers="[{value:0,label:'Floor',color:'gray'}]" :min="-0.3" :max="0.6" axis="Δ − Δ_floor（bits/byte）" :digits="3" />
</div>

<!--
看 Fake-French 的翻譯增益，中文方向一樣。

[click] 理解端，也就是 Fake-French 翻英文，綁定的核心實驗是 +0.386。

[互動] 輸出頭換成隨機的，理解端會受影響嗎？

[click] 隨機輸出頭 +0.447、加線性層 +0.414，都不輸核心實驗，跟 seed 噪音同一個量級。中文是 +0.267 和 +0.215，對 +0.242。所以理解不需要綁定。

[click] 生成端，英文翻 Fake-French，核心實驗是 +0.484。

[click] 隨機輸出頭掉到 −0.181，比 Floor 還低，中文也是 −0.104。這在設計上可以預期：沒有 L_ban 資料，獨立輸出頭學不到那些列。加線性層是 +0.357，比綁定差 0.127，大約是 seed 噪音的三倍，效果遠不如推論期置中。這兩個子實驗都只有一個 seed。
-->

---
layout: statement
---

<ResultsExpTag exp="untie" />

# 理解不需要 *tie*

生成需要的是：輸出端有預對齊的列

<!--
所以依事先判準，這組我們判定為不確定，但讀法很清楚：tie 的必要性可能沒有想像中大。理解端不需要綁定；生成端需要的是輸出端用到預對齊的列，tie 只是在零樣本設定下「免費」拿到這些列的手段。

結果部分到這裡。所有結論都是 likelihood 層面的，實際生成的 off-target rate 還沒量。接下來把這些結果收斂成幾個結論。
-->
