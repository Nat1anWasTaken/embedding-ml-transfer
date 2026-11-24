<script setup lang="ts">
import { computed } from 'vue'
import { useSlideContext } from '@slidev/client'

// How the shuffle control is built and why it is evidence.
// Click 1: L_ban labels swap onto other L_ban vectors (vectors stay put) and the
// procedure steps appear. Click 2: shuffle penalty, core run vs collapsed run.
const { $clicks } = useSlideContext()

// Seen L_train vectors and the L_ban vector aligned next to each one.
const pairs = [
  { en: 'museum', zh: '博物館', x: 92, y: 92 },
  { en: 'water', zh: '水', x: 262, y: 74 },
  { en: 'mountain', zh: '山', x: 86, y: 224 },
  { en: 'fire', zh: '火', x: 268, y: 214 },
]
// Where each L_ban token lands after the shuffle (index into pairs).
const perm = [2, 3, 1, 0]
const shuffled = computed(() => $clicks.value >= 1)

function zhPos(i: number) {
  const p = pairs[shuffled.value ? perm[i] : i]
  return `translate(${p.x + 30}px, ${p.y + 22}px)`
}

const steps = [
  '同一個 checkpoint，不重訓',
  'L<tspan baseline-shift="sub" style="font-size:10px">ban</tspan> 列組內重排（中文 28,677 · Fake 29,622）',
  'L<tspan baseline-shift="sub" style="font-size:10px">train</tspan> 列不動，輸入輸出一起換（tied）',
  '同一批 FLORES+ 算 BPB，seed 0–4 平均',
]

const bars = [
  { label: '核心 · 中文', value: 0.624, color: 'var(--google-red)' },
  { label: '核心 · Fake', value: 1.482, color: 'var(--google-green)' },
  { label: '崩潰 · 中文', value: 0.0001, color: 'var(--google-gray)' },
  { label: '崩潰 · Fake', value: 0.0018, color: 'var(--google-gray)' },
]
const barMax = 1.6
const barW = 250
</script>

<template>
  <svg class="shuffle-proc" viewBox="0 0 820 340" role="img">
    <!-- Embedding plane: what the body sees is only positions -->
    <rect class="plane" x="10" y="20" width="350" height="290" rx="14" />
    <text class="plane-title" x="26" y="46">embedding 空間（示意）</text>
    <g v-for="p in pairs" :key="p.en">
      <circle class="en-dot" :cx="p.x" :cy="p.y" r="7" />
      <text class="en-label" :x="p.x - 12" :y="p.y - 12">{{ p.en }}</text>
      <circle class="zh-dot" :cx="p.x + 30" :cy="p.y + 22" r="7" />
    </g>
    <g v-for="(p, i) in pairs" :key="p.zh" class="zh-label" :class="{ moved: shuffled }" :style="{ transform: zhPos(i) }">
      <text x="12" y="6">{{ p.zh }}</text>
    </g>
    <g class="legend">
      <circle class="en-dot" cx="30" cy="292" r="6" /><text x="42" y="297">主體看過（L<tspan baseline-shift="sub" class="sub">train</tspan>）</text>
      <circle class="zh-dot" cx="190" cy="292" r="6" /><text x="202" y="297">沒看過（L<tspan baseline-shift="sub" style="font-size:10px">ban</tspan>）</text>
    </g>

    <!-- Procedure -->
    <g class="fade" :class="{ on: $clicks >= 1 }">
      <text class="head" x="400" y="40">打亂對照怎麼做</text>
      <g v-for="(s, i) in steps" :key="i" :transform="`translate(400, ${72 + i * 30})`">
        <circle class="step-dot" cx="6" cy="-5" r="4" />
        <text class="step" x="20" y="0" v-html="s" />
      </g>
    </g>

    <!-- Evidence -->
    <g class="fade" :class="{ on: $clicks >= 2 }">
      <text class="head" x="400" y="214">打亂後變差多少（語言內 BPB）</text>
      <g v-for="(b, i) in bars" :key="b.label" :transform="`translate(400, ${236 + i * 26})`">
        <text class="bar-label" x="86" y="13" text-anchor="end">{{ b.label }}</text>
        <rect class="track" x="98" y="2" :width="barW" height="14" rx="7" />
        <rect class="bar" x="98" y="2" :width="$clicks >= 2 ? Math.max(3, (b.value / barMax) * barW) : 3" height="14" rx="7" :style="{ fill: b.color }" />
        <text class="bar-val" :x="98 + Math.max(3, (b.value / barMax) * barW) + 8" y="14" :style="{ fill: b.color }">+{{ b.value < 0.01 ? b.value.toFixed(4) : b.value.toFixed(3) }}</text>
      </g>
    </g>
  </svg>
</template>

<style scoped>
.shuffle-proc { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.plane { fill: var(--google-surface); stroke: var(--google-border); stroke-width: 1.5; }
.plane-title { fill: var(--google-muted); font-size: 15px; font-weight: 500; }
.en-dot { fill: var(--google-blue); }
.zh-dot { fill: var(--google-red); }
.en-label { fill: var(--google-blue); font-size: 16px; font-weight: 600; text-anchor: end; }
.zh-label text { fill: var(--google-red); font-size: 17px; font-weight: 700; }
.zh-label { transition: transform 900ms cubic-bezier(0.6, 0, 0.2, 1); }
.legend text { fill: var(--google-muted); font-size: 14px; }
.sub { font-size: 10px; }
.fade { opacity: 0; transition: opacity 400ms ease; }
.fade.on { opacity: 1; }
.head { fill: var(--google-ink); font-size: 18px; font-weight: 700; }
.step-dot { fill: var(--google-blue); }
.step { fill: var(--google-ink); font-size: 15.5px; }
.bar-label { fill: var(--google-ink); font-size: 14.5px; font-weight: 500; }
.track { fill: var(--google-surface); }
.bar { transition: width 800ms cubic-bezier(0.2, 0.8, 0.2, 1); }
.bar-val { font-size: 14px; font-weight: 600; }
</style>
