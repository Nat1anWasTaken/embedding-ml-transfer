<script setup lang="ts">
import { useSlideContext } from '@slidev/client'

// Three ways to fill the L_ban rows of E: Floor (all = mean), shuffled
// (distinct vectors, wrong token), raw (distinct, right token).
// Panel 0 is visible from the start; panels 1 and 2 appear on clicks 1 and 2.
const { $clicks } = useSlideContext()

const tokens = ['貓', '狗', '水', '火', '山']
const colors = ['var(--google-blue)', 'var(--google-red)', 'var(--google-yellow)', 'var(--google-green)', '#a142f4']
const angles = [-40, -20, 0, 20, 40]
const perm = [2, 4, 0, 1, 3]

const panels = [
  { title: 'Floor', note: '全部 = 平均向量', kind: 'floor' },
  { title: '打亂', note: '彼此不同、對應錯', kind: 'shuffle' },
  { title: '原始', note: '彼此不同、對應對', kind: 'raw' },
] as const

function arrow(kind: string, i: number) {
  if (kind === 'floor')
    return { angle: 0, color: 'var(--google-gray)' }
  const k = kind === 'shuffle' ? perm[i] : i
  return { angle: angles[k], color: colors[k] }
}
function end(angle: number) {
  const r = (angle * Math.PI) / 180
  return { x: 52 * Math.cos(r), y: 30 * Math.sin(r) }
}
</script>

<template>
  <svg class="rows-sketch" viewBox="0 0 820 330" role="img">
    <g v-for="(p, pi) in panels" :key="p.kind" :transform="`translate(${pi * 280}, 0)`">
    <g :class="['panel', { on: $clicks >= pi }]">
      <rect class="card" x="6" y="6" width="252" height="318" rx="16" />
      <text class="title" x="132" y="44" text-anchor="middle">{{ p.title }}</text>
      <g v-for="(t, i) in tokens" :key="t" :transform="`translate(0, ${78 + i * 46})`">
        <rect class="chip" x="40" y="-17" width="38" height="34" rx="8" :style="{ stroke: colors[i] }" />
        <text class="tok" x="59" y="8" text-anchor="middle">{{ t }}</text>
        <text class="eq" x="100" y="7" text-anchor="middle">→</text>
        <g transform="translate(150, 0)">
          <line x1="0" y1="0" :x2="end(arrow(p.kind, i).angle).x" :y2="end(arrow(p.kind, i).angle).y" :style="{ stroke: arrow(p.kind, i).color }" class="vec" />
          <circle :cx="end(arrow(p.kind, i).angle).x" :cy="end(arrow(p.kind, i).angle).y" r="5" :style="{ fill: arrow(p.kind, i).color }" />
        </g>
      </g>
      <text class="note" x="132" y="312" text-anchor="middle">{{ p.note }}</text>
    </g>
    </g>
  </svg>
</template>

<style scoped>
.rows-sketch { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.panel { opacity: 0; transform: translateY(10px); transition: opacity 400ms ease, transform 500ms cubic-bezier(0.2, 0.8, 0.2, 1); }
.panel.on { opacity: 1; transform: none; }
.card { fill: var(--google-surface); stroke: var(--google-border); stroke-width: 1.5; }
.title { fill: var(--google-ink); font-size: 22px; font-weight: 700; }
.chip { fill: white; stroke-width: 3; }
.tok { fill: var(--google-ink); font-size: 19px; font-weight: 600; }
.eq { fill: var(--google-muted); font-size: 18px; }
.vec { stroke-width: 4; stroke-linecap: round; }
.note { fill: var(--google-muted); font-size: 17px; font-weight: 500; }
</style>
