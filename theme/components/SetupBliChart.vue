<script setup lang="ts">
/**
 * G-Emb gate: CSLS P@1 vs. the shuffled-candidates random baseline (max of 5), log axis.
 * Row i appears at step >= row.at.
 */
interface Row { label: string, sub: string, p: number, ci: [number, number], rand: number, color: string, at: number }

withDefaults(defineProps<{ step?: number }>(), { step: 99 })

const rows: Row[] = [
  { label: 'Oracle_Emb', sub: 'en–zh · 4,417 對', p: 0.4840, ci: [0.4693, 0.4988], rand: 0.00023, color: 'var(--google-red)', at: 1 },
  { label: 'Oracle_Emb', sub: 'Fake-French · 29,574 對', p: 0.7997, ci: [0.7951, 0.8042], rand: 0.000034, color: 'var(--google-green)', at: 2 },
  { label: 'Qwen3-0.6B-Base', sub: 'en–zh', p: 0.5257, ci: [0.5110, 0.5404], rand: 0, color: 'var(--google-blue)', at: 3 },
  { label: 'Qwen3-1.7B-Base', sub: 'en–zh', p: 0.5225, ci: [0.5078, 0.5372], rand: 0, color: 'var(--google-blue)', at: 3 },
]

const L = 250
const R = 790
function x(v: number) {
  return L + ((Math.log10(v) + 5) / 5) * (R - L)
}
const ticks = [1e-5, 1e-4, 1e-3, 1e-2, 1e-1, 1]
const tickLabel = ['10⁻⁵', '10⁻⁴', '10⁻³', '0.01', '0.1', '1']
const rowH = 58
const top = 20
</script>

<template>
  <svg class="setup-bli" :viewBox="`0 0 820 ${top + rows.length * rowH + 50}`" role="img">
    <g class="grid">
      <g v-for="(t, i) in ticks" :key="t">
        <line :x1="x(t)" :x2="x(t)" :y1="top - 6" :y2="top + rows.length * rowH" />
        <text :x="x(t)" :y="top + rows.length * rowH + 22" text-anchor="middle">{{ tickLabel[i] }}</text>
      </g>
      <text :x="(L + R) / 2" :y="top + rows.length * rowH + 46" text-anchor="middle" class="axis">P@1（log 軸）</text>
    </g>
    <g v-for="(r, i) in rows" :key="i" :transform="`translate(0, ${top + i * rowH + rowH / 2})`" :class="['row', { on: step >= r.at }]">
      <text class="lab" :x="L - 16" y="-2" text-anchor="end">{{ r.label }}</text>
      <text class="sub" :x="L - 16" y="17" text-anchor="end">{{ r.sub }}</text>
      <template v-if="r.rand > 0">
        <circle class="rand" :cx="x(r.rand)" cy="0" r="6" />
        <text class="rand-t" :x="x(r.rand)" y="-12" text-anchor="middle">隨機最大 {{ r.rand }}</text>
        <line class="gap" :x1="x(r.rand) + 8" :x2="x(r.p) - 10" y1="0" y2="0" :style="{ stroke: r.color }" />
      </template>
      <text v-else class="rand-t" :x="x(1e-5) + 4" y="5">隨機基準 = 0</text>
      <circle :cx="x(r.p)" cy="0" r="9" :style="{ fill: r.color }" />
      <text class="val" :x="x(r.p) - 14" y="-13" text-anchor="end" :style="{ fill: r.color }">{{ r.p.toFixed(3) }}</text>
    </g>
  </svg>
</template>

<style scoped>
.setup-bli { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.grid line { stroke: var(--google-border); stroke-width: 1; stroke-dasharray: 3 4; }
.grid text { fill: var(--google-muted); font-size: 14px; }
.grid .axis { font-size: 15px; }
.row { opacity: 0; transition: opacity 350ms ease; }
.row.on { opacity: 1; }
.lab { fill: var(--google-ink); font-size: 18px; font-weight: 600; }
.sub { fill: var(--google-muted); font-size: 14px; }
.rand { fill: var(--google-gray); }
.rand-t { fill: var(--google-muted); font-size: 13px; }
.gap { stroke-width: 3; opacity: 0.35; stroke-dasharray: 2 5; }
.val { font-size: 17px; font-weight: 700; }
</style>
