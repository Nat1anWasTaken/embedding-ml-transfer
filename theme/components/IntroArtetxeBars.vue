<script setup lang="ts">
import { useSlideContext } from '@slidev/client'

// Artetxe, Ruder & Yogatama (ACL 2020), Tables 1–3: averages over all languages incl. English.
// CLWE = 768d ident (their best CLWE variant); MonoTrans = token emb (their basic variant).
const tasks = [
  { name: 'MLDoc', note: '文件分類', metric: 'acc', clwe: 82.3, mono: 81.2 },
  { name: 'XNLI', note: '自然語言推論', metric: 'acc', clwe: 60.1, mono: 67.8 },
  { name: 'XQuAD', note: '抽取式 QA', metric: 'F1', clwe: 39.5, mono: 61.1 },
]

// The slide sets `clicks: 4`: one click per task, then the English-only reference.
const { $clicks } = useSlideContext()

const W = 760
const left = 150
const barH = 22
const groupH = 86
const scale = (v: number) => (v / 100) * (W - left - 70)
</script>

<template>
  <div class="artetxe-bars">
    <div class="legend">
      <span><i class="clwe" />CLWE（對照組）</span>
      <span><i class="mono" />MonoTrans（作者的方法）</span>
    </div>
    <svg :viewBox="`0 0 ${W} ${tasks.length * groupH + 10}`" role="img">
      <g v-for="(t, i) in tasks" :key="t.name" :transform="`translate(0, ${i * groupH + 8})`" :class="{ on: $clicks >= i + 1 }" class="group">
        <text class="task" :x="left - 16" y="22" text-anchor="end">{{ t.name }}</text>
        <text class="note" :x="left - 16" y="44" text-anchor="end">{{ t.note }}</text>
        <rect class="clwe" :x="left" y="4" :height="barH" rx="3" :width="$clicks >= i + 1 ? scale(t.clwe) : 0" />
        <text class="val clwe-t" :x="left + scale(t.clwe) + 8" y="21">{{ t.clwe.toFixed(1) }}</text>
        <rect class="mono" :x="left" :y="8 + barH" :height="barH" rx="3" :width="$clicks >= i + 1 ? scale(t.mono) : 0" />
        <text class="val mono-t" :x="left + scale(t.mono) + 8" :y="25 + barH">{{ t.mono.toFixed(1) }}</text>
      </g>
    </svg>
    <div class="caption">
      <span>全語言平均（含英文）· 由簡單到困難</span>
      <b :class="{ on: $clicks >= 4 }">只看英文：XNLI 82.4 vs 83.1，XQuAD 84.2 vs 83.9</b>
    </div>
  </div>
</template>

<style scoped>
.artetxe-bars { width: 100%; }
svg { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }

.legend { display: flex; gap: 1.6rem; margin-bottom: 0.6rem; color: var(--google-muted); font-size: 15px; }
.legend i { display: inline-block; width: 0.8rem; height: 0.8rem; margin-right: 0.4rem; border-radius: 2px; vertical-align: -1px; }
i.clwe, rect.clwe { background: var(--google-red); fill: var(--google-red); }
i.mono, rect.mono { background: var(--google-blue); fill: var(--google-blue); }

rect { transition: width 700ms cubic-bezier(0.2, 0.8, 0.2, 1); }
.group .val { opacity: 0; transition: opacity 300ms ease 400ms; }
.group.on .val { opacity: 1; }
.group .task, .group .note { opacity: 0.35; transition: opacity 300ms ease; }
.group.on .task, .group.on .note { opacity: 1; }

.task { fill: var(--google-ink); font-size: 19px; font-weight: 600; }
.note { fill: var(--google-muted); font-size: 14px; }
.val { font-size: 16px; font-weight: 600; }
.clwe-t { fill: var(--google-red); }
.mono-t { fill: var(--google-blue); }

.caption { display: flex; justify-content: space-between; margin-top: 0.4rem; color: var(--google-muted); font-size: 14px; }
.caption b { opacity: 0; color: var(--google-ink); font-weight: 600; transition: opacity 300ms ease; }
.caption b.on { opacity: 1; }
</style>
