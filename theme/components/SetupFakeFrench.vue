<script setup lang="ts">
/**
 * How Fake-French is built (schematic tokenisation, not real Qwen3 IDs).
 * step 1: French tokenised with Qwen3; step 2: every token in S shifted to 151,936 + idx(t);
 * step 3: shared anchors (EOS, digits, whitespace, ASCII punctuation) stay unshifted.
 */
withDefaults(defineProps<{ step?: number }>(), { step: 99 })

const toks = [
  { t: '<|endoftext|>', anchor: true },
  { t: 'Le', anchor: false },
  { t: '␣chat', anchor: false },
  { t: '␣dort', anchor: false },
  { t: '␣', anchor: true },
  { t: '3', anchor: true },
  { t: '␣heures', anchor: false },
  { t: '.', anchor: true },
]
</script>

<template>
  <div class="ff">
    <div class="ff-row">
      <div class="ff-label">法文</div>
      <div class="ff-text">Le chat dort 3 heures.</div>
    </div>

    <div class="tok-grid">
      <div class="ff-label" :class="{ hide: step < 1 }">Qwen3 切詞</div>
      <span v-for="(k, i) in toks" :key="`o${i}`" class="chip orig" :class="{ hide: step < 1, anchor: k.anchor && step >= 3 }">{{ k.t }}</span>

      <div class="ff-label" />
      <span v-for="(k, i) in toks" :key="`a${i}`" class="arrow" :class="{ hide: step < 2, keep: k.anchor && step >= 3 }">{{ k.anchor && step >= 3 ? '‖' : '↓' }}</span>

      <div class="ff-label" :class="{ hide: step < 2 }">Fake-French</div>
      <span
        v-for="(k, i) in toks" :key="`s${i}`"
        class="chip"
        :class="[k.anchor && step >= 3 ? 'anchor' : 'shifted', { hide: step < 2 }]"
      >{{ k.t }}<sup v-if="!(k.anchor && step >= 3)">′</sup></span>
    </div>

    <div class="ff-rule" :class="{ hide: step < 2 }">
      t <span class="map">→</span> 151,936 + idx(t)
      <span class="rule-note">字串一樣、ID 全新：主體從沒看過這些列</span>
    </div>
    <div class="ff-rule anchor-rule" :class="{ hide: step < 3 }">
      錨點不平移：EOS · 阿拉伯數字 · 空白 · ASCII 標點
      <span class="rule-note">和真中文跟英文共用的是同一類錨點</span>
    </div>
  </div>
</template>

<style scoped>
.ff { display: flex; flex-direction: column; gap: 0.55rem; font-family: var(--deck-font-sans); }
.ff-row { display: grid; grid-template-columns: 8.5rem 1fr; align-items: center; transition: opacity 350ms ease; }
.hide { opacity: 0; }
.ff-label { color: var(--google-muted); font-size: 16px; text-align: right; padding-right: 1rem; }
.ff-text { font-size: 26px; font-weight: 500; }
.tok-grid { display: grid; grid-template-columns: 8.5rem repeat(8, auto); gap: 0.35rem 0.4rem; justify-content: start; align-items: center; }
.tok-grid > * { transition: opacity 350ms ease; }
.chip {
  padding: 0.2rem 0.55rem; border-radius: 0.5rem; border: 2px solid; text-align: center;
  font-family: var(--slidev-code-font-family); font-size: 16px; white-space: nowrap;
  transition: all 350ms ease;
}
.chip.orig { border-color: #e37400; background: #fef3e6; color: #a05000; }
.chip.shifted { border-color: var(--google-green); background: #e6f4ea; color: var(--google-green-dark); font-weight: 600; }
.chip.anchor { border-color: var(--google-gray); background: var(--google-surface); color: var(--google-muted); }
.chip sup { font-size: 12px; }
.arrow { text-align: center; color: var(--google-green); font-size: 18px; line-height: 1; }
.arrow.keep { color: var(--google-gray); }
.ff-rule { margin-left: 8.5rem; margin-top: 0.3rem; font-size: 20px; font-weight: 600; color: var(--google-green-dark); transition: opacity 350ms ease; }
.ff-rule .map { color: var(--google-muted); padding: 0 0.2rem; }
.ff-rule.anchor-rule { color: var(--google-muted); margin-top: 0; }
.rule-note { display: block; font-size: 15px; font-weight: 400; color: var(--google-muted); }
</style>
