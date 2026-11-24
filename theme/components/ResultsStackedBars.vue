<script setup lang="ts">
import { useSlideContext } from '@slidev/client'

type Color = 'blue' | 'red' | 'yellow' | 'green' | 'gray'

interface Part {
  value: number
  color?: Color
}

interface Row {
  label: string
  parts: Part[]
  /** Shown as the row total; pass the source's own total so rounding never drifts. */
  total?: number
  at?: number
}

const props = withDefaults(defineProps<{
  rows: Row[]
  max?: number
  /** Legend entries, one per part index. */
  legend?: { label: string, color?: Color }[]
  digits?: number
}>(), {
  max: 6,
  legend: () => [],
  digits: 3,
})

// The slide needs `clicks: <n>` in its frontmatter.
const { $clicks } = useSlideContext()

const W = 820
const left = 150
const right = 80
const rowH = 58
const top = 46

function w(v: number) {
  return (v / props.max) * (W - left - right)
}
function offset(row: Row, j: number) {
  return row.parts.slice(0, j).reduce((s, p) => s + w(p.value), 0)
}
function sum(row: Row) {
  return row.parts.reduce((s, p) => s + p.value, 0)
}
function shown(row: Row, i: number) {
  return $clicks.value >= (row.at ?? i + 1)
}
</script>

<template>
  <svg class="stacked" :viewBox="`0 0 ${W} ${top + props.rows.length * rowH + 10}`" role="img">
    <g v-for="(l, j) in props.legend" :key="l.label" :class="l.color ?? 'blue'" :transform="`translate(${left + j * 190}, 14)`">
      <rect class="swatch" width="18" height="18" rx="4" />
      <text class="legend" x="26" y="15">{{ l.label }}</text>
    </g>
    <g v-for="(row, i) in props.rows" :key="row.label" :transform="`translate(0, ${top + i * rowH})`" :class="{ on: shown(row, i) }">
      <text class="row-label" :x="left - 16" :y="rowH / 2 + 3" text-anchor="end">{{ row.label }}</text>
      <rect class="track" :x="left" y="10" :width="W - left - right" :height="rowH - 22" rx="6" />
      <g v-for="(p, j) in row.parts" :key="j" :class="['part', p.color ?? 'blue']">
        <rect
          class="bar"
          :x="left + offset(row, j)"
          y="10"
          :width="w(p.value)"
          :height="rowH - 22"
          :style="{ transitionDelay: `${j * 250}ms` }"
        />
        <text
          v-if="w(p.value) > 52"
          class="seg"
          :x="left + offset(row, j) + w(p.value) / 2"
          :y="rowH / 2 + 3"
          text-anchor="middle"
          :style="{ transitionDelay: `${j * 250 + 300}ms` }"
        >{{ p.value.toFixed(props.digits) }}</text>
        <text
          v-else
          class="seg outside"
          :x="left + offset(row, j) + w(p.value) + 8"
          :y="rowH / 2 + 3"
          :style="{ transitionDelay: `${j * 250 + 300}ms` }"
        >+{{ p.value.toFixed(props.digits) }}</text>
      </g>
      <text v-if="row.total !== undefined" class="total" :x="W - right + 12" :y="rowH / 2 + 3">{{ row.total.toFixed(props.digits) }}</text>
    </g>
  </svg>
</template>

<style scoped>
.stacked { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }

.blue { --c: var(--google-blue); }
.red { --c: var(--google-red); }
.yellow { --c: var(--google-yellow); }
.green { --c: var(--google-green); }
.gray { --c: var(--google-gray); }

.swatch { fill: var(--c); }
.legend { fill: var(--google-ink); font-size: 16px; }
.row-label { fill: var(--google-ink); font-size: 19px; font-weight: 500; }
.track { fill: var(--google-surface); }

.bar {
  fill: var(--c);
  transform-box: fill-box;
  transform-origin: left center;
  transform: scaleX(0);
  transition: transform 700ms cubic-bezier(0.2, 0.8, 0.2, 1);
}
.on .bar { transform: scaleX(1); }
.seg { fill: white; font-size: 16px; font-weight: 600; opacity: 0; transition: opacity 300ms ease; }
.seg.outside { fill: var(--c); }
.on .seg { opacity: 1; }
.total { fill: var(--google-muted); font-size: 17px; font-weight: 600; opacity: 0; transition: opacity 300ms ease 700ms; }
.on .total { opacity: 1; }
</style>
