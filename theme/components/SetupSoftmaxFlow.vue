<script setup lang="ts">
/**
 * One training step of the frozen-E tied model, drawn end to end:
 * token → E (vertical: rows = token IDs) → body → h → Eᵀ → softmax p_v.
 * Same row colouring as SetupEMatrix, rotated 90°. Bars are schematic.
 *
 * step 0: forward pass (a pulse runs through the model)
 * step 1: the target row y gets pulled up (−E_y term)
 * step 2: every L_ban row is in the sum as a negative (+Σ p_v E_v term)
 * step 3: rows can't move, so the body learns to steer h away → L_ban bars collapse
 */
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

const V = 181632
const Y0 = 52
const Y1 = 300
function y(id: number) {
  return Y0 + (id / V) * (Y1 - Y0)
}

// Deterministic pseudo-random positions, as in SetupEMatrix.
function scatter(n: number, seed: number, lo: number, hi: number) {
  const out: number[] = []
  let s = seed
  for (let i = 0; i < n; i++) {
    s = (s * 9301 + 49297) % 233280
    out.push(lo + ((i + s / 233280) / n) * (hi - lo))
  }
  return out
}
// Fewer, thinner stripes than the horizontal band so the column stays readable.
const zh = scatter(24, 7, 300, 151300)

const INPUT_ID = 42000
const TARGET_ID = 97000

// Output distribution: one bar per sampled row.
interface Bar { id: number, kind: 'train' | 'zh' | 'fake' | 'target', len: number }
const bars: Bar[] = []
let s = 11
function rnd() {
  s = (s * 9301 + 49297) % 233280
  return s / 233280
}
for (let id = 2500; id < 151000; id += 6200) {
  if (Math.abs(id - TARGET_ID) < 4000) continue
  bars.push({ id, kind: 'train', len: 14 + rnd() * 40 })
}
for (const id of zh.filter((_, i) => i % 2 === 1))
  bars.push({ id, kind: 'zh', len: 18 + rnd() * 34 })
for (let id = 154000; id < 181000; id += 4200)
  bars.push({ id, kind: 'fake', len: 18 + rnd() * 34 })
bars.push({ id: TARGET_ID, kind: 'target', len: 150 })

function barLen(b: Bar) {
  if (b.kind === 'target')
    return props.step >= 1 ? b.len : 60
  if (b.kind !== 'train' && props.step >= 3)
    return 4
  return b.len
}

const isBan = (b: Bar) => b.kind === 'zh' || b.kind === 'fake'
</script>

<template>
  <svg class="setup-softmax" viewBox="0 0 860 360" role="img" :class="`s${props.step}`">
    <defs>
      <marker id="sf-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
        <path d="M0,0 L10,5 L0,10 z" fill="#9aa0a6" />
      </marker>
    </defs>

    <!-- tie arc between the two copies of E -->
    <path class="tie" d="M92,46 C92,4 552,4 552,46" />
    <text class="tie-label" x="322" y="20" text-anchor="middle">同一個矩陣（tied）· 凍結</text>

    <!-- input token -->
    <text class="io" x="48" :y="y(INPUT_ID) + 6" text-anchor="end">museum</text>
    <line x1="50" :y1="y(INPUT_ID)" x2="58" :y2="y(INPUT_ID)" class="flow" />

    <!-- E (input side), rows = token IDs top → bottom -->
    <g class="emat">
      <rect class="frame" x="60" :y="Y0" width="64" :height="Y1 - Y0" rx="4" />
      <rect class="ltrain" x="60" :y="Y0" width="64" :height="y(151643) - Y0" />
      <rect v-for="(id, i) in zh" :key="`l${i}`" class="zh" x="60" :y="y(id)" width="64" height="1.6" />
      <rect class="fake" x="60" :y="y(151936)" width="64" :height="y(181558) - y(151936)" />
      <rect class="row-hit" x="56" :y="y(INPUT_ID) - 3" width="72" height="7" rx="2" />
      <text class="mat-label" x="92" y="326" text-anchor="middle">E</text>
      <text class="sub" x="92" y="346" text-anchor="middle">查 museum 那一列</text>
    </g>

    <line x1="132" :y1="y(INPUT_ID)" x2="178" y2="176" class="flow" marker-end="url(#sf-arrow)" />

    <!-- body -->
    <g class="body" :class="{ learning: props.step >= 3 }">
      <rect v-for="i in 7" :key="i" :x="182 + (i - 1) * 34" y="104" width="26" height="144" rx="5" />
      <text class="sub" x="300" y="274" text-anchor="middle">主體 · 訓練中</text>
      <text v-if="props.step >= 3" class="learn-label" x="300" y="94" text-anchor="middle">學會讓 h 避開 L<tspan dy="4" class="subs">ban</tspan><tspan dy="-4"> 列</tspan></text>
    </g>

    <line x1="420" y1="176" x2="452" y2="176" class="flow" marker-end="url(#sf-arrow)" />
    <circle class="h" cx="468" cy="176" r="14" />
    <text class="h-label" x="468" y="182" text-anchor="middle">h</text>
    <line x1="484" y1="176" x2="516" y2="176" class="flow" marker-end="url(#sf-arrow)" />
    <text class="sub" x="500" y="206" text-anchor="middle">E h</text>

    <!-- Eᵀ (output side) -->
    <g class="emat">
      <rect class="frame" x="520" :y="Y0" width="64" :height="Y1 - Y0" rx="4" />
      <rect class="ltrain" x="520" :y="Y0" width="64" :height="y(151643) - Y0" />
      <rect v-for="(id, i) in zh" :key="`r${i}`" class="zh" :class="{ neg: props.step === 2 }" x="520" :y="y(id)" width="64" height="1.6" />
      <rect class="fake" :class="{ neg: props.step === 2 }" x="520" :y="y(151936)" width="64" :height="y(181558) - y(151936)" />
      <rect v-if="props.step >= 1" class="row-target" x="516" :y="y(TARGET_ID) - 3" width="72" height="7" rx="2" />
      <text class="mat-label" x="552" y="326" text-anchor="middle">Eᵀ</text>
      <text class="sub" x="552" y="346" text-anchor="middle">每一列和 h 內積</text>
    </g>

    <!-- softmax output p_v -->
    <g class="bars">
      <line x1="606" :y1="Y0" x2="606" :y2="Y1" class="axis" />
      <rect
        v-for="(b, i) in bars" :key="`b${i}`"
        :class="['bar', b.kind, { neg: isBan(b) && props.step === 2 }]"
        x="607" :y="y(b.id) - 2" :width="barLen(b)" height="4" rx="2"
      />
      <text class="mat-label small" x="690" y="326" text-anchor="middle">softmax → p<tspan dy="4" class="subs">v</tspan></text>
      <text class="sub" x="690" y="346" text-anchor="middle">整個詞表（示意）</text>

      <g v-if="props.step >= 1" class="tag target-tag">
        <text x="766" :y="y(TARGET_ID) + 5">y = opens ↑</text>
      </g>
      <g v-if="props.step >= 2" class="tag neg-tag">
        <text x="690" :y="y(151936) - 18">L<tspan dy="4" class="subs">ban</tspan><tspan dy="-4">：永遠不是答案</tspan></text>
        <text x="690" :y="y(151936) - 2">每一步都被往下壓 ↓</text>
      </g>
    </g>

    <!-- forward-pass pulse -->
    <circle class="pulse" r="5">
      <animateMotion
        dur="2.6s" repeatCount="indefinite"
        :path="`M50,${y(INPUT_ID)} L92,${y(INPUT_ID)} L178,176 L420,176 L468,176 L552,176 L640,${y(TARGET_ID)}`"
      />
    </circle>
  </svg>
</template>

<style scoped>
.setup-softmax { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.frame { fill: none; stroke: var(--google-gray); stroke-width: 2; stroke-dasharray: 6 4; }
.ltrain { fill: var(--google-blue); opacity: 0.55; }
.zh { fill: var(--google-red); }
.fake { fill: var(--google-green); }
.row-hit { fill: none; stroke: var(--google-ink); stroke-width: 2.5; }
.row-target { fill: none; stroke: var(--google-blue-dark); stroke-width: 3; }
.flow { stroke: var(--google-gray); stroke-width: 2.5; fill: none; }
.io { fill: var(--google-ink); font-size: 17px; font-weight: 600; }
.mat-label { fill: var(--google-ink); font-size: 22px; font-weight: 700; }
.mat-label.small { font-size: 18px; }
.sub { fill: var(--google-muted); font-size: 14px; }
.subs { font-size: 0.7em; }
.tie { fill: none; stroke: var(--google-blue); stroke-width: 2.2; stroke-dasharray: 7 5; }
.tie-label { fill: var(--google-blue-dark); font-size: 15px; font-weight: 600; stroke: white; stroke-width: 6px; paint-order: stroke; }

.body rect { fill: var(--google-surface); stroke: var(--google-border); stroke-width: 2; transition: all 500ms ease; }
.body.learning rect { fill: #fce8e6; stroke: var(--google-red); }
.learn-label { fill: var(--google-red); font-size: 16px; font-weight: 600; }

.h { fill: var(--google-blue-soft); stroke: var(--google-blue); stroke-width: 2.5; }
.h-label { fill: var(--google-blue-dark); font-size: 17px; font-weight: 700; font-style: italic; }

.axis { stroke: var(--google-border); stroke-width: 1.5; }
.bar { transition: width 1200ms cubic-bezier(0.2, 0.8, 0.2, 1), opacity 400ms ease; }
.bar.train { fill: var(--google-blue); opacity: 0.45; }
.bar.zh { fill: var(--google-red); }
.bar.fake { fill: var(--google-green); }
.bar.target { fill: var(--google-blue-dark); }

.tag text { font-size: 15px; font-weight: 600; }
.target-tag text { fill: var(--google-blue-dark); }
.neg-tag text { fill: var(--google-red); }

.neg { animation: neg-pulse 0.9s ease-in-out infinite alternate; }
@keyframes neg-pulse { from { opacity: 1; } to { opacity: 0.3; } }

.pulse { fill: var(--google-yellow); stroke: white; stroke-width: 1.5; }
</style>
