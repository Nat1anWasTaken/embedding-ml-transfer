<script setup lang="ts">
/**
 * Δ (translation gain): score the same reference y after the real source x vs. an unrelated x′.
 * step 1: second row (x′), step 2: Δ formula, step 3: the two directions.
 */
withDefaults(defineProps<{ step?: number }>(), { step: 99 })
</script>

<template>
  <div class="sd">
    <div class="sd-row">
      <span class="blk shot">3-shot 範例</span>
      <span class="blk src">x　真正的來源句</span>
      <span class="blk tgt">y　參考譯文 <i>← 只對這段計分</i></span>
      <span class="res">BPB(y | x)</span>
    </div>
    <div class="sd-row" :class="{ hide: step < 1 }">
      <span class="blk shot">3-shot 範例</span>
      <span class="blk src alt">x′　長度相近、不相干的句子</span>
      <span class="blk tgt">y　參考譯文 <i>← 只對這段計分</i></span>
      <span class="res">BPB(y | x′)</span>
    </div>
    <div class="sd-formula" :class="{ hide: step < 2 }">
      Δ = BPB(y | x′) − BPB(y | x)
      <small>來源句真的有幫上忙多少 bits/byte</small>
    </div>
    <div class="sd-dirs" :class="{ hide: step < 3 }">
      <div><b>L<sub>ban</sub> → En</b>　輸出是英文，繞過 tied 輸出端<span>量「讀懂」：理解端</span></div>
      <div><b>En → L<sub>ban</sub></b>　輸出要走 L<sub>ban</sub> 的列<span>量「寫出來」：生成端</span></div>
    </div>
  </div>
</template>

<style scoped>
.sd { display: flex; flex-direction: column; gap: 0.7rem; font-family: var(--deck-font-sans); }
.sd > * { transition: opacity 350ms ease; }
.hide { opacity: 0; }
.sd-row { display: flex; align-items: center; gap: 0.4rem; }
.blk { padding: 0.45rem 0.8rem; border-radius: 0.6rem; font-size: 17px; white-space: nowrap; }
.blk i { font-style: normal; font-size: 14px; opacity: 0.8; }
.shot { background: var(--google-surface); border: 1.5px solid var(--google-border); color: var(--google-muted); }
.src { background: var(--google-blue-soft); border: 1.5px solid var(--google-blue); color: var(--google-blue-dark); font-weight: 600; min-width: 13.5rem; }
.src.alt { background: #f1f3f4; border-style: dashed; border-color: var(--google-gray); color: var(--google-muted); }
.tgt { background: #fce8e6; border: 1.5px solid var(--google-red); color: #a50e0e; font-weight: 600; }
.res { margin-left: 0.6rem; font-size: 17px; color: var(--google-ink); font-family: var(--slidev-code-font-family); }
.sd-formula { margin-top: 0.5rem; font-size: 28px; font-weight: 700; color: var(--google-ink); }
.sd-formula small { display: block; font-size: 16px; font-weight: 400; color: var(--google-muted); }
.sd-dirs { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; margin-top: 0.4rem; }
.sd-dirs div { padding: 0.7rem 1rem; border-radius: 0.8rem; background: var(--google-surface); font-size: 16px; color: var(--google-muted); }
.sd-dirs b { color: var(--google-ink); font-size: 20px; }
.sd-dirs span { display: block; color: var(--google-blue-dark); font-weight: 600; font-size: 17px; margin-top: 0.2rem; }
</style>
