<script setup lang="ts">
/**
 * G-Ruler gate: Δ (L_ban → En, FLORES+ 3-shot) for Floor vs. Upper, and gap / CI width (pass ≥ 5).
 * Row i appears at step >= i + 1.
 */
withDefaults(defineProps<{ step?: number }>(), { step: 99 })

const rows = [
  { label: 'zh → en', floor: 0.0288, upper: 0.4474, ratio: 24.3, color: 'var(--google-red)' },
  { label: 'fake → en', floor: 0.0021, upper: 0.6453, ratio: 24.0, color: 'var(--google-green)' },
  { label: 'fr → en', floor: 0.2019, upper: 0.3484, ratio: 14.9, color: '#e37400' },
]
const L = 130
const R = 620
const MAX = 0.7
function x(v: number) { return L + (v / MAX) * (R - L) }
const rowH = 70
const top = 40
const ticks = [0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7]
</script>

<template>
  <svg class="setup-ruler" :viewBox="`0 0 820 ${top + rows.length * rowH + 50}`" role="img">
    <text :x="R + 100" y="22" text-anchor="middle" class="head">差距 ÷ CI 寬度</text>
    <g class="grid">
      <g v-for="t in ticks" :key="t">
        <line :x1="x(t)" :x2="x(t)" :y1="top - 8" :y2="top + rows.length * rowH" />
        <text :x="x(t)" :y="top + rows.length * rowH + 22" text-anchor="middle">{{ t.toFixed(1) }}</text>
      </g>
      <text :x="(L + R) / 2" :y="top + rows.length * rowH + 46" text-anchor="middle" class="axis">Δ（bits/byte）</text>
    </g>
    <g v-for="(r, i) in rows" :key="r.label" :transform="`translate(0, ${top + i * rowH + rowH / 2})`" :class="['row', { on: step >= i + 1 }]">
      <text class="lab" :x="L - 16" y="6" text-anchor="end">{{ r.label }}</text>
      <line :x1="x(r.floor)" :x2="x(r.upper)" y1="0" y2="0" class="span" :style="{ stroke: r.color }" />
      <circle :cx="x(r.floor)" cy="0" r="8" class="floor" />
      <text :x="x(r.floor)" y="-14" text-anchor="middle" class="t-floor">Floor {{ r.floor.toFixed(3) }}</text>
      <circle :cx="x(r.upper)" cy="0" r="9" :style="{ fill: r.color }" />
      <text :x="x(r.upper)" y="-14" text-anchor="middle" class="t-up" :style="{ fill: r.color }">Upper {{ r.upper.toFixed(3) }}</text>
      <text :x="R + 100" y="8" text-anchor="middle" class="ratio">{{ r.ratio.toFixed(1) }}</text>
    </g>
    <text :x="R + 100" :y="top + rows.length * rowH + 22" text-anchor="middle" class="th" :opacity="step >= 1 ? 1 : 0">門檻 ≥ 5</text>
  </svg>
</template>

<style scoped>
.setup-ruler { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.head { fill: var(--google-muted); font-size: 15px; font-weight: 600; }
.grid line { stroke: var(--google-border); stroke-width: 1; stroke-dasharray: 3 4; }
.grid text { fill: var(--google-muted); font-size: 14px; }
.grid .axis { font-size: 15px; }
.row { opacity: 0; transition: opacity 350ms ease; }
.row.on { opacity: 1; }
.lab { fill: var(--google-ink); font-size: 19px; font-weight: 600; }
.span { stroke-width: 6; stroke-linecap: round; opacity: 0.35; }
.floor { fill: var(--google-gray); }
.t-floor { fill: var(--google-muted); font-size: 13px; }
.t-up { font-size: 13px; font-weight: 600; }
.ratio { fill: var(--google-ink); font-size: 30px; font-weight: 700; }
.th { fill: var(--google-muted); font-size: 15px; }
</style>
