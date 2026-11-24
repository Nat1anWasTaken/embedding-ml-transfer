<script setup lang="ts">
/**
 * −log p(y) = −log p(y | y ∈ V_L) − log P(V_L), drawn on the model's next-token distribution.
 * Proportions are schematic.
 * step 1: the language set V_L, 2: 語言選擇 bracket, 3: 語言內排序 bracket.
 */
withDefaults(defineProps<{ step?: number }>(), { step: 99 })

const X0 = 60
const X1 = 780
// schematic: V_L holds ~55% of the mass, y holds ~30% of V_L
const vl = 0.55
const yIn = 0.3
const xVL = X0 + (X1 - X0) * (1 - vl)
const xY0 = xVL + 26
const xY1 = xY0 + (X1 - xVL) * yIn
</script>

<template>
  <svg class="setup-decomp" viewBox="0 0 820 250" role="img">

    <!-- whole distribution -->
    <rect class="other" :x="X0" y="70" :width="X1 - X0" height="56" rx="6" />
    <g :class="['vl', { on: step >= 1 }]">
      <rect :x="xVL" y="70" :width="X1 - xVL" height="56" rx="6" />
      <text :x="(xVL + X1) / 2 + 60" y="104" text-anchor="middle">V<tspan class="subs">L</tspan>：這個語言的 token</text>
    </g>
    <g :class="['y', { on: step >= 3 }]">
      <rect :x="xY0" y="76" :width="xY1 - xY0" height="44" rx="5" />
      <text :x="(xY0 + xY1) / 2" y="104" text-anchor="middle">y</text>
    </g>
    <text :x="(X0 + xVL) / 2" y="104" text-anchor="middle" class="other-t" :opacity="step >= 1 ? 1 : 0">其他語言的 token</text>

    <!-- 語言選擇 bracket: V_L share of the whole -->
    <g :class="['br', 'sel', { on: step >= 2 }]">
      <path :d="`M${X0},140 v10 H${X1} v-10`" />
      <path :d="`M${xVL},150 v10 H${X1} v-10`" class="inner" />
      <text :x="(X0 + X1) / 2" y="186" text-anchor="middle">語言選擇　−log P(V<tspan class="subs">L</tspan>)：機率有沒有放在這個語言上</text>
    </g>

    <!-- 語言內排序 bracket: y share inside V_L -->
    <g :class="['br', 'within', { on: step >= 3 }]">
      <path :d="`M${xVL},62 v-10 H${X1} v10`" />
      <path :d="`M${xY0},52 v-10 H${xY1} v10`" class="inner" />
      <text :x="(xY0 + xY1) / 2" y="30" text-anchor="middle">語言內排序　−log p(y | V<tspan class="subs">L</tspan>)</text>
    </g>
    <text v-if="step >= 3" :x="(X0 + X1) / 2" y="228" text-anchor="middle" class="hint">在這組 token 裡有沒有排對 → 直接檢驗幾何有沒有被用上</text>
  </svg>
</template>

<style scoped>
.setup-decomp { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.cap { fill: var(--google-muted); font-size: 15px; }
.other { fill: var(--google-surface); stroke: var(--google-border); stroke-width: 1.5; }
.other-t { fill: var(--google-gray); font-size: 16px; transition: opacity 350ms; }
.vl, .y, .br { opacity: 0; transition: opacity 350ms ease; }
.on { opacity: 1; }
.vl rect { fill: #fce8e6; stroke: var(--google-red); stroke-width: 2; }
.vl text { fill: var(--google-red); font-size: 17px; font-weight: 600; }
.y rect { fill: var(--google-red); }
.y text { fill: white; font-size: 20px; font-weight: 700; font-style: italic; }
.br path { fill: none; stroke: var(--google-gray); stroke-width: 2; }
.br path.inner { stroke: var(--c); stroke-width: 2.5; }
.br text { fill: var(--c); font-size: 17px; font-weight: 600; }
.sel { --c: var(--google-muted); }
.within { --c: var(--google-red); }
.subs { font-size: 12px; baseline-shift: sub; }
.hint { fill: var(--google-ink); font-size: 16px; }
</style>
