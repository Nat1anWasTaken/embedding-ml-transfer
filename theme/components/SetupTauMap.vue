<script setup lang="ts">
/**
 * A BPB axis drawn with the exact x-geometry of RevealChart (W 820, left 190, right 40)
 * for min −0.2 / max 1, so it sits directly above the τ line and maps BPB → τ.
 * Example numbers: 真中文 語言內排序 BPB, Floor 4.530, Upper (B0-Upper) 1.457.
 */
withDefaults(defineProps<{ step?: number }>(), { step: 99 })

const F = 4.530
const U = 1.457
const MIN = -0.2
const MAX = 1
const W = 820
const left = 190
const right = 40
function xt(t: number) { return left + ((t - MIN) / (MAX - MIN)) * (W - left - right) }
function xb(b: number) { return xt((F - b) / (F - U)) }
const ticks = [5, 4, 3, 2]
</script>

<template>
  <svg class="setup-taumap" :viewBox="`0 0 ${W} 120`" role="img">
    <g :class="['ax', { on: step >= 1 }]">
      <text class="name" :x="left - 18" y="58" text-anchor="end">真中文 BPB</text>
      <line :x1="left" :x2="W - right" y1="52" y2="52" class="axis" />
      <g v-for="t in ticks" :key="t">
        <line :x1="xb(t)" :x2="xb(t)" y1="46" y2="58" class="tick" />
        <text :x="xb(t)" y="78" text-anchor="middle" class="tick-t">{{ t }}</text>
      </g>
      <circle :cx="xb(F)" cy="52" r="8" class="floor" />
      <text :x="xb(F)" y="30" text-anchor="middle" class="floor-t">Floor {{ F.toFixed(3) }}</text>
      <circle :cx="xb(U)" cy="52" r="8" class="upper" />
      <text :x="xb(U)" y="30" text-anchor="middle" class="upper-t">Upper {{ U.toFixed(3) }}</text>
      <text :x="xb(3)" y="100" text-anchor="middle" class="dir">BPB 越低越好 →</text>
    </g>
    <g :class="['conn', { on: step >= 2 }]">
      <line :x1="xb(F)" :x2="xt(0)" y1="62" y2="120" />
      <line :x1="xb(U)" :x2="xt(1)" y1="62" y2="120" />
    </g>
  </svg>
</template>

<style scoped>
.setup-taumap { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.ax, .conn { opacity: 0; transition: opacity 350ms ease; }
.on { opacity: 1; }
.name { fill: var(--google-ink); font-size: 17px; font-weight: 500; }
.axis { stroke: var(--google-border); stroke-width: 1.5; }
.tick { stroke: var(--google-gray); stroke-width: 1.5; }
.tick-t { fill: var(--google-muted); font-size: 14px; }
.floor { fill: var(--google-gray); }
.upper { fill: var(--google-blue); }
.floor-t { fill: var(--google-muted); font-size: 15px; font-weight: 600; }
.upper-t { fill: var(--google-blue); font-size: 15px; font-weight: 600; }
.dir { fill: var(--google-muted); font-size: 13px; }
.conn line { stroke: var(--google-gray); stroke-width: 1.5; stroke-dasharray: 4 4; }
</style>
