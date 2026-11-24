<script setup lang="ts">
import { computed } from 'vue'

/**
 * The tied embedding E drawn as a horizontal band: x = token ID (0 → 181,632),
 * height = hidden dim. Stripes are schematic (V_ban rows are scattered across
 * the base BPE range in Qwen3's vocab), group sizes are to scale.
 *
 * mode:
 *  - lang     : L_train rows / 真中文 V_ban rows / Fake-French rows
 *  - oracle   : same, labelled as Oracle_Emb (every row trained by B0-Upper)
 *  - floor    : first Floor definition — V_ban + all rows ≥ 151,936 set to the mean μ
 *  - grad     : full softmax — L_ban rows act as negatives every step
 *  - mask     : training-time softmax mask — 67,701 rows outside the keep set are dropped
 *  - floorall : current Floor — all 67,701 masked rows set to μ
 */
const props = withDefaults(defineProps<{
  mode?: 'lang' | 'oracle' | 'floor' | 'grad' | 'mask' | 'floorall'
  step?: number
}>(), { mode: 'lang', step: 99 })

const V = 181632
const X0 = 40
const X1 = 780
const Y0 = 58
const Y1 = 168
function x(id: number) {
  return X0 + (id / V) * (X1 - X0)
}

// Deterministic pseudo-random positions (schematic only).
function scatter(n: number, seed: number, lo: number, hi: number) {
  const out: number[] = []
  let s = seed
  for (let i = 0; i < n; i++) {
    s = (s * 9301 + 49297) % 233280
    const r = s / 233280
    out.push(lo + ((i + r) / n) * (hi - lo))
  }
  return out
}
// 28,677 / 151,643 ≈ 19% of the base range → 58 stripes of ~2.4px.
const zh = scatter(58, 7, 300, 151300)
// 9,037 base tokens never seen in Multi-Train (≈6%) → 22 thin stripes.
const unseen = scatter(22, 31, 900, 150500).map(v => v + 1700)

const muZh = computed(() => props.mode === 'floor' || props.mode === 'floorall')
const muFake = computed(() => props.mode === 'floor' || props.mode === 'floorall')
const muUnseen = computed(() => props.mode === 'floorall')
const dropped = computed(() => props.mode === 'mask')
const showUnseen = computed(() => props.mode === 'mask' || props.mode === 'floorall')
</script>

<template>
  <svg class="setup-ematrix" viewBox="0 0 820 260" role="img" :class="`m-${props.mode}`">
    <defs>
      <pattern id="mu-hatch" width="6" height="6" patternUnits="userSpaceOnUse" patternTransform="rotate(45)">
        <rect width="6" height="6" fill="#ede7f6" />
        <line x1="0" y1="0" x2="0" y2="6" stroke="#9575cd" stroke-width="2.4" />
      </pattern>
    </defs>

    <!-- ID ticks -->
    <g class="ticks">
      <text :x="x(0)" y="40" text-anchor="start">0</text>
      <text :x="x(151643) - 4" y="40" text-anchor="end">151,643</text>
      <text :x="x(181632)" y="40" text-anchor="end">181,632</text>
      <text x="410" y="18" text-anchor="middle" class="axis-name">token ID →</text>
    </g>
    <text class="dim" x="26" :y="(Y0 + Y1) / 2" text-anchor="middle" :transform="`rotate(-90 26 ${(Y0 + Y1) / 2})`">d = 1024</text>

    <!-- base BPE rows (L_train) -->
    <rect class="ltrain" :x="x(0)" :y="Y0" :width="x(151643) - x(0)" :height="Y1 - Y0" rx="3" />

    <!-- unseen-in-Multi-Train base tokens (only matter for the mask / Floor) -->
    <g v-if="showUnseen">
      <rect
        v-for="(id, i) in unseen" :key="`u${i}`"
        :class="muUnseen ? 'mu' : (dropped ? 'dropped gray' : 'gray')"
        :x="x(id)" :y="Y0" width="1.8" :height="Y1 - Y0"
      />
    </g>

    <!-- 真中文 V_ban rows, scattered -->
    <rect
      v-for="(id, i) in zh" :key="`z${i}`"
      :class="muZh ? 'mu' : (dropped ? 'dropped zh' : ['zh', { neg: props.mode === 'grad' && props.step >= 4 }])"
      :x="x(id)" :y="Y0" width="2.4" :height="Y1 - Y0"
    />

    <!-- special / padding rows -->
    <rect
      :class="muFake || props.mode === 'floorall' ? 'mu' : (dropped ? 'dropped gray' : 'gray')"
      :x="x(151643)" :y="Y0" width="3" :height="Y1 - Y0"
    />

    <!-- Fake-French rows 151,936–181,557 -->
    <rect
      :class="muFake ? 'mu' : (dropped ? 'dropped fake' : ['fake', { neg: props.mode === 'grad' && props.step >= 4 }])"
      :x="x(151936)" :y="Y0" :width="x(181558) - x(151936)" :height="Y1 - Y0" rx="3"
    />
    <rect class="gray" :class="{ mu: props.mode === 'floor' || props.mode === 'floorall', dropped }" :x="x(181558)" :y="Y0" width="2" :height="Y1 - Y0" />

    <!-- labels per mode -->
    <g class="labels">
      <template v-if="props.mode === 'lang' || props.mode === 'oracle' || props.mode === 'grad'">
        <g v-if="props.step >= 1">
          <rect class="ltrain" x="40" y="196" width="16" height="16" rx="3" />
          <text x="62" y="210">L<tspan class="subs">train</tspan> 共用列</text>
        </g>
        <g v-if="props.step >= 2">
          <rect class="zh" x="230" y="196" width="16" height="16" rx="3" />
          <text x="252" y="210">真中文 V<tspan class="subs">ban</tspan>　28,677 列（散落）</text>
        </g>
        <g v-if="props.step >= 3">
          <rect class="fake" x="540" y="196" width="16" height="16" rx="3" />
          <text x="562" y="210">Fake-French　29,622 列</text>
        </g>
      </template>
      <template v-if="props.mode === 'floor'">
        <rect class="mu" x="40" y="196" width="16" height="16" rx="3" />
        <text x="62" y="210">L<tspan class="subs">ban</tspan> 列 → 全部設成已訓練列的平均 μ</text>
      </template>
      <template v-if="props.mode === 'mask'">
        <rect class="ltrain" x="40" y="196" width="16" height="16" rx="3" />
        <text x="62" y="210">保留 113,931 列（訓練集出現過）</text>
        <rect class="dropped zh" x="400" y="196" width="16" height="16" rx="3" />
        <text x="422" y="210">排除 67,701 列：不進 logsumexp</text>
      </template>
      <template v-if="props.mode === 'floorall'">
        <rect class="mu" x="40" y="196" width="16" height="16" rx="3" />
        <text x="62" y="210">遮罩外 67,701 列全部 → μ</text>
        <text x="420" y="210" class="muted">（含 Multi-Train 沒出現的 9,037 個 base token）</text>
      </template>
      <text v-if="props.mode === 'oracle' && props.step >= 4" x="780" y="244" text-anchor="end" class="strong">Oracle_Emb　[181,632 × 1024]　每一列都訓練過</text>
      <text v-if="props.mode === 'grad' && props.step >= 4" x="410" y="244" text-anchor="middle" class="neg-label">完整 softmax：每一步 L<tspan class="subs">ban</tspan> 列都是負例</text>
    </g>
  </svg>
</template>

<style scoped>
.setup-ematrix {
  width: 100%;
  overflow: visible;
  font-family: var(--deck-font-sans);
}
.ticks text { fill: var(--google-muted); font-size: 14px; }
.ticks .axis-name { font-size: 15px; }
.dim { fill: var(--google-muted); font-size: 14px; }
.ltrain { fill: var(--google-blue); opacity: 0.55; }
.zh { fill: var(--google-red); }
.fake { fill: var(--google-green); }
.gray { fill: var(--google-gray); }
.mu { fill: url(#mu-hatch); opacity: 1; }
.dropped { opacity: 0.16; }
.neg { animation: neg-pulse 1.2s ease-in-out infinite alternate; }
@keyframes neg-pulse { from { opacity: 1; } to { opacity: 0.35; } }
.labels text { fill: var(--google-ink); font-size: 17px; }
.subs { font-size: 12px; baseline-shift: sub; }
.labels .muted { fill: var(--google-muted); font-size: 15px; }
.labels .strong { font-weight: 600; fill: var(--google-blue-dark); }
.labels .neg-label { font-weight: 600; fill: var(--google-red); }
</style>
