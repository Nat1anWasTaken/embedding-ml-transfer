<script setup lang="ts">
import { useSlideContext } from '@slidev/client'

type Color = 'blue' | 'red' | 'yellow' | 'green' | 'gray'

interface Series {
  label: string
  /** [x (training progress %), y] pairs. */
  points: [number, number][]
  color?: Color
  /** Click index that draws this series; defaults to its position (1, 2, ...). */
  at?: number
  dashed?: boolean
  /** Vertical nudge for the end label, in chart units. */
  labelDy?: number
}

const props = withDefaults(defineProps<{
  series: Series[]
  yMax?: number
  xLabel?: string
  yLabel?: string
}>(), {
  yMax: 1,
  xLabel: '訓練進度',
  yLabel: '',
})

// The slide needs `clicks: <n>` in its frontmatter.
const { $clicks } = useSlideContext()

const W = 820
const H = 330
const left = 70
const right = 190
const top = 24
const bottom = 52

function x(v: number) {
  return left + (v / 100) * (W - left - right)
}
function y(v: number) {
  return top + (1 - v / props.yMax) * (H - top - bottom)
}
function path(s: Series) {
  return s.points.map((p, i) => `${i ? 'L' : 'M'}${x(p[0])},${y(p[1])}`).join(' ')
}
function shown(s: Series, i: number) {
  return $clicks.value >= (s.at ?? i + 1)
}
const yTicks = [0, 0.25, 0.5, 0.75, 1]
const xTicks = [0, 25, 50, 75, 100]
</script>

<template>
  <svg class="drift-chart" :viewBox="`0 0 ${W} ${H}`" role="img">
    <g v-for="t in yTicks" :key="`y${t}`">
      <line class="grid" :x1="left" :x2="W - right" :y1="y(t * props.yMax)" :y2="y(t * props.yMax)" />
      <text class="tick" :x="left - 12" :y="y(t * props.yMax) + 5" text-anchor="end">{{ (t * props.yMax).toFixed(2) }}</text>
    </g>
    <g v-for="t in xTicks" :key="`x${t}`">
      <text class="tick" :x="x(t)" :y="H - bottom + 24" text-anchor="middle">{{ t }}%</text>
    </g>
    <line class="axis" :x1="left" :x2="W - right" :y1="H - bottom" :y2="H - bottom" />
    <text class="axis-title" :x="(left + W - right) / 2" :y="H - 4" text-anchor="middle">{{ props.xLabel }}</text>
    <text v-if="props.yLabel" class="axis-title" :x="left" :y="top - 8" text-anchor="start">{{ props.yLabel }}</text>

    <g v-for="(s, i) in props.series" :key="s.label" :class="['series', s.color ?? 'blue', { on: shown(s, i), dashed: s.dashed }]">
      <path class="line" :d="path(s)" pathLength="1" />
      <circle v-for="p in s.points" :key="p[0]" class="pt" :cx="x(p[0])" :cy="y(p[1])" r="5" />
      <text class="end-label" :x="x(s.points[s.points.length - 1][0]) + 14" :y="y(s.points[s.points.length - 1][1]) + 6 + (s.labelDy ?? 0)">{{ s.label }}</text>
    </g>
  </svg>
</template>

<style scoped>
.drift-chart { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }

.blue { --c: var(--google-blue); }
.red { --c: var(--google-red); }
.yellow { --c: var(--google-yellow); }
.green { --c: var(--google-green); }
.gray { --c: var(--google-gray); }

.grid { stroke: var(--google-surface); stroke-width: 2; }
.axis { stroke: var(--google-border); stroke-width: 1.5; }
.tick { fill: var(--google-muted); font-size: 15px; }
.axis-title { fill: var(--google-muted); font-size: 16px; }

.series .line {
  fill: none;
  stroke: var(--c);
  stroke-width: 4;
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-dasharray: 1;
  stroke-dashoffset: 1;
  transition: stroke-dashoffset 1100ms cubic-bezier(0.2, 0.8, 0.2, 1);
}
.series.on .line { stroke-dashoffset: 0; }
.series.dashed .line { stroke-dasharray: 0.012 0.012; stroke-dashoffset: 0; opacity: 0; transition: opacity 500ms ease; }
.series.dashed.on .line { opacity: 1; }
.series .pt, .series .end-label { opacity: 0; transition: opacity 400ms ease 500ms; }
.series.on .pt, .series.on .end-label { opacity: 1; }
.pt { fill: white; stroke: var(--c); stroke-width: 3; }
.end-label { fill: var(--c); font-size: 17px; font-weight: 600; }
</style>
