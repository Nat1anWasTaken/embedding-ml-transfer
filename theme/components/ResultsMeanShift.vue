<script setup lang="ts">
import { computed } from 'vue'
import { useSlideContext } from '@slidev/client'

// Schematic (not data): the L_ban rows sit as a cloud offset from the L_train
// cloud; mean-centering translates the whole L_ban cloud by
// μ(L_train) − μ(L_ban), keeping its internal shape.
const props = withDefaults(defineProps<{
  /** Click index at which the L_ban cloud shifts; 0 = already shifted. */
  shiftAt?: number
  /** Hide the text labels (for a small corner copy). */
  bare?: boolean
}>(), {
  shiftAt: 1,
  bare: false,
})

const { $clicks } = useSlideContext()
const shifted = computed(() => $clicks.value >= props.shiftAt)

function cloud(seed: number, n: number, sx: number, sy: number) {
  let s = seed
  const rnd = () => {
    s = (s * 1103515245 + 12345) % 2147483648
    return s / 2147483648
  }
  return Array.from({ length: n }, () => {
    // Box–Muller for a roundish cloud.
    const u = Math.max(rnd(), 1e-6)
    const v = rnd()
    const r = Math.sqrt(-2 * Math.log(u))
    return { x: r * Math.cos(2 * Math.PI * v) * sx, y: r * Math.sin(2 * Math.PI * v) * sy }
  })
}

const train = { cx: 300, cy: 200 }
const ban = { cx: 590, cy: 90 }
const trainPts = cloud(7, 70, 62, 42)
const banPts = cloud(42, 40, 40, 26)
const dx = train.cx - ban.cx
const dy = train.cy - ban.cy
</script>

<template>
  <svg class="mean-shift" viewBox="0 0 820 300" role="img">
    <defs>
      <marker id="ms-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
        <path d="M0,0 L10,5 L0,10 z" class="arrow-head" />
      </marker>
    </defs>
    <g :transform="`translate(${train.cx}, ${train.cy})`">
      <circle v-for="(p, i) in trainPts" :key="`t${i}`" :cx="p.x" :cy="p.y" r="5" class="pt train" />
      <circle r="9" class="mu train-mu" />
      <text v-if="!props.bare" class="label train-label" x="0" y="96" text-anchor="middle">L_train 的列</text>
    </g>
    <line
      v-if="!props.bare"
      :class="['shift-arrow', { gone: shifted }]"
      :x1="ban.cx - 14" :y1="ban.cy + 6" :x2="train.cx + 18" :y2="train.cy - 6"
      marker-end="url(#ms-arrow)"
    />
    <text v-if="!props.bare" :class="['label', 'formula', { gone: shifted }]" x="470" y="178" text-anchor="start">μ(L_train) − μ(L_ban)</text>
    <g class="ban-cloud" :style="{ transform: shifted ? `translate(${ban.cx + dx}px, ${ban.cy + dy}px)` : `translate(${ban.cx}px, ${ban.cy}px)` }">
      <circle v-for="(p, i) in banPts" :key="`b${i}`" :cx="p.x" :cy="p.y" r="5" class="pt ban" />
      <circle r="9" class="mu ban-mu" />
      <text v-if="!props.bare" class="label ban-label" x="0" y="-48" text-anchor="middle">L_ban 的列</text>
    </g>
  </svg>
</template>

<style scoped>
.mean-shift { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.pt.train { fill: var(--google-gray); opacity: 0.45; }
.pt.ban { fill: var(--google-blue); opacity: 0.8; }
.mu { stroke: white; stroke-width: 3; }
.train-mu { fill: var(--google-ink); }
.ban-mu { fill: var(--google-blue-dark); }
.ban-cloud { transition: transform 1000ms cubic-bezier(0.2, 0.8, 0.2, 1); }
.label { font-size: 19px; font-weight: 600; }
.train-label { fill: var(--google-muted); }
.ban-label { fill: var(--google-blue); }
.formula { fill: var(--google-ink); font-size: 18px; transition: opacity 300ms ease; }
.shift-arrow { stroke: var(--google-ink); stroke-width: 2.5; stroke-dasharray: 6 6; transition: opacity 300ms ease; }
.arrow-head { fill: var(--google-ink); }
.gone { opacity: 0; }
</style>
