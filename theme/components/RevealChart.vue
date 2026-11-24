<script setup lang="ts">
import { computed } from 'vue'
import { useSlideContext } from '@slidev/client'

interface Row {
  label: string
  value: number
  /** 95% CI as [low, high], drawn as a whisker. */
  ci?: [number, number]
  color?: 'blue' | 'red' | 'yellow' | 'green' | 'gray'
  /** Click index that reveals this row; defaults to its position (1, 2, ...). */
  at?: number
}

interface Marker {
  value: number
  label: string
  color?: 'blue' | 'red' | 'yellow' | 'green' | 'gray'
}

const props = withDefaults(defineProps<{
  rows: Row[]
  min?: number
  max?: number
  /** Reference lines drawn from the start, e.g. Floor at 0 and Upper at 1. */
  markers?: Marker[]
  /** Axis title under the chart, e.g. "τ". */
  axis?: string
  /** Decimal places in value labels. */
  digits?: number
  /** Width reserved for row labels (SVG units). */
  labelWidth?: number
}>(), {
  min: 0,
  max: 1,
  markers: () => [{ value: 0, label: 'Floor', color: 'gray' }, { value: 1, label: 'Upper', color: 'blue' }],
  digits: 2,
  labelWidth: 190,
})

// The slide needs `clicks: <rows>` in its frontmatter so these steps exist.
const { $clicks } = useSlideContext()

const W = 820
const left = props.labelWidth
const right = 40
const rowH = 54
const top = 34
const height = computed(() => top + props.rows.length * rowH + 44)

function x(v: number) {
  return left + ((v - props.min) / (props.max - props.min)) * (W - left - right)
}

function shown(row: Row, i: number) {
  return $clicks.value >= (row.at ?? i + 1)
}
</script>

<template>
  <svg class="reveal-chart" :viewBox="`0 0 ${W} ${height}`" role="img">
    <g v-for="m in props.markers" :key="m.label" :class="['marker', m.color ?? 'gray']">
      <line :x1="x(m.value)" :x2="x(m.value)" :y1="top - 10" :y2="height - 38" />
      <text :x="x(m.value)" :y="top - 16" text-anchor="middle">{{ m.label }}</text>
    </g>
    <line class="axis" :x1="left" :x2="W - right" :y1="height - 38" :y2="height - 38" />
    <text v-if="props.axis" class="axis-title" :x="(left + W - right) / 2" :y="height - 8" text-anchor="middle">{{ props.axis }}</text>
    <g v-for="(row, i) in props.rows" :key="row.label" :transform="`translate(0, ${top + i * rowH + rowH / 2})`">
      <text class="row-label" :x="left - 18" y="7" text-anchor="end">{{ row.label }}</text>
      <line class="track" :x1="left" :x2="W - right" y1="0" y2="0" />
      <g :class="['datum', row.color ?? 'blue', { on: shown(row, i) }]">
        <line v-if="row.ci" class="ci" :x1="x(row.ci[0])" :x2="x(row.ci[1])" y1="0" y2="0" />
        <g class="dot" :style="{ transform: `translateX(${shown(row, i) ? x(row.value) : x(props.min)}px)` }">
          <circle r="9" />
          <text y="-16" text-anchor="middle">{{ row.value.toFixed(props.digits) }}</text>
        </g>
      </g>
    </g>
  </svg>
</template>

<style scoped>
.reveal-chart {
  width: 100%;
  overflow: visible;
  font-family: var(--deck-font-sans);
}

.blue { --c: var(--google-blue); }
.red { --c: var(--google-red); }
.yellow { --c: var(--google-yellow); }
.green { --c: var(--google-green); }
.gray { --c: var(--google-gray); }

.marker line { stroke: var(--c); stroke-width: 2; stroke-dasharray: 5 5; }
.marker text { fill: var(--c); font-size: 15px; font-weight: 600; }
.axis { stroke: var(--google-border); stroke-width: 1.5; }
.axis-title { fill: var(--google-muted); font-size: 17px; }
.row-label { fill: var(--google-ink); font-size: 19px; font-weight: 500; }
.track { stroke: var(--google-surface); stroke-width: 10; stroke-linecap: round; }

.datum { opacity: 0; transition: opacity 300ms ease; }
.datum.on { opacity: 1; }
.ci { stroke: var(--c); stroke-width: 4; stroke-linecap: round; opacity: 0.35; }
.dot { transition: transform 800ms cubic-bezier(0.2, 0.8, 0.2, 1); }
.dot circle { fill: var(--c); }
.dot text { fill: var(--c); font-size: 16px; font-weight: 600; }
</style>
