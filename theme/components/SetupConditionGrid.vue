<script setup lang="ts">
/**
 * The three L_ban conditions as a 2×2: token overlap (new / seen) × language distance (near / far).
 * step 1: 真法文, 2: Fake-French, 3: 真中文 (+ the empty cell), 4: overlap bracket, 5: distance bracket.
 * The Fake-French cell carries view-transition-name `cond-fake` so it can morph into the next slide.
 */
withDefaults(defineProps<{ step?: number }>(), { step: 99 })
</script>

<template>
  <div class="cond-grid">
    <div class="col-h" style="grid-area: 1 / 2">token 全新<small>主體訓練時沒看過這些 ID</small></div>
    <div class="col-h" style="grid-area: 1 / 3">token 見過<small>子詞和英／西／德大量共用</small></div>

    <div class="row-h" style="grid-area: 2 / 1">語言<b>近</b></div>
    <div class="cell fake" :class="{ on: step >= 2 }" style="grid-area: 2 / 2; view-transition-name: cond-fake">
      <b>Fake-French</b>
      <span>法文 token ID 平移成新 ID</span>
      <em>主體訓練時禁用</em>
    </div>
    <div class="cell fr" style="grid-area: 2 / 3" :class="{ on: step >= 1 }">
      <b>真法文</b>
      <span>fra_Latn</span>
      <em>主體訓練時也沒有</em>
    </div>

    <div class="row-h" style="grid-area: 3 / 1">語言<b>遠</b></div>
    <div class="cell zh" style="grid-area: 3 / 2" :class="{ on: step >= 3 }">
      <b>真中文</b>
      <span>cmn_Hani · V<sub>ban</sub> 28,677 列</span>
      <em>主體訓練時禁用</em>
    </div>
    <div class="cell empty" style="grid-area: 3 / 3" :class="{ on: step >= 3 }">
      <span>（沒有這一格）</span>
    </div>

    <div class="cg-ol cg-ol-row" :class="{ on: step >= 4 }" />
    <div class="cg-ol cg-ol-col" :class="{ on: step >= 5 }" />
    <div class="legend">
      <span class="lg" :class="{ on: step >= 4 }">上排：Fake-French vs 真法文 → 只差 <b>token 重疊</b></span>
      <span class="lg" :class="{ on: step >= 5 }">左欄：Fake-French vs 真中文 → 只差 <b>語言距離</b></span>
    </div>
  </div>
</template>

<style scoped>
.cond-grid {
  position: relative;
  display: grid;
  grid-template-columns: 7rem 17rem 17rem;
  grid-template-rows: auto 7.6rem 7.6rem auto;
  gap: 0.8rem;
  font-family: var(--deck-font-sans);
}
.col-h { display: flex; flex-direction: column; align-items: center; font-size: 20px; font-weight: 600; }
.col-h small { color: var(--google-muted); font-size: 14px; font-weight: 400; }
.row-h { display: flex; flex-direction: column; justify-content: center; align-items: flex-end; padding-right: 0.4rem; color: var(--google-muted); font-size: 17px; }
.row-h b { color: var(--google-ink); font-size: 30px; line-height: 1.1; }
.cell {
  display: flex; flex-direction: column; justify-content: center; gap: 0.15rem;
  padding: 0.8rem 1.1rem; border-radius: 1rem; border: 2px solid var(--c, var(--google-border));
  background: color-mix(in srgb, var(--c, white) 9%, white);
  opacity: 0; transform: translateY(6px); transition: opacity 350ms ease, transform 350ms ease;
}
.cell.on { opacity: 1; transform: none; }
.cell b { font-size: 26px; color: var(--c); line-height: 1.15; }
.cell span { font-size: 16px; color: var(--google-ink); }
.cell em { font-style: normal; font-size: 14px; color: var(--google-muted); }
.cell.fake { --c: var(--google-green-dark); }
.cell.fr { --c: #e37400; }
.cell.zh { --c: var(--google-red); }
.cell.empty { --c: var(--google-border); border-style: dashed; background: var(--google-surface); align-items: center; }
.cell.empty span { color: var(--google-gray); }
.cg-ol { pointer-events: none; margin: -0.35rem; border-radius: 1.25rem; border: 3px dashed var(--google-blue); opacity: 0; transition: opacity 350ms ease; }
.cg-ol.on { opacity: 1; }
.cg-ol-row { grid-row: 2; grid-column: 2 / 4; }
.cg-ol-col { grid-row: 2 / 4; grid-column: 2; border-color: var(--google-blue-dark); border-style: dotted; }
.legend { grid-row: 4; grid-column: 2 / 4; display: flex; flex-direction: column; align-items: flex-start; gap: 0.1rem; margin-top: 0.4rem; font-size: 16px; color: var(--google-muted); }
.lg { white-space: nowrap; opacity: 0; transition: opacity 350ms ease; }
.lg.on { opacity: 1; }
.lg b { color: var(--google-blue-dark); }
</style>
