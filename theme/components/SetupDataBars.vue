<script setup lang="ts">
/**
 * The three training sets, to scale (billions of Qwen3 tokens).
 * step 1: Mono-Train, step 2: Oracle-Train's Multi part, step 3: + 中文 + Fake-French.
 */
withDefaults(defineProps<{
  step?: number
  /** `oracle` draws only the full Oracle-Train bar, as a compact recap. */
  only?: 'oracle'
}>(), { step: 99 })

const X0 = 170
const PER_B = 24 // px per billion tokens → 24.24B ≈ 582px
function w(b: number) { return b * PER_B }

const multi = [
  { k: 'en', b: 8.08, cls: 'en' },
  { k: 'es', b: 4.04, cls: 'es' },
  { k: 'de', b: 4.04, cls: 'de' },
  { k: 'ru', b: 4.04, cls: 'ru' },
]
function offs(i: number) {
  return multi.slice(0, i).reduce((s, m) => s + w(m.b), 0)
}
</script>

<template>
  <svg v-if="only === 'oracle'" class="setup-data" viewBox="0 0 820 56" role="img">
    <g transform="translate(0, 4)">
      <text class="name" :x="X0 - 16" y="30" text-anchor="end">Oracle-Train</text>
      <g v-for="(m, i) in multi" :key="m.k">
        <rect :class="m.cls" :x="X0 + offs(i)" y="4" :width="w(m.b) - 2" height="40" rx="4" />
        <text class="seg" :x="X0 + offs(i) + w(m.b) / 2" y="30" text-anchor="middle">{{ m.k }}</text>
      </g>
      <rect class="zh" :x="X0 + w(20.2)" y="4" :width="w(2.02) - 2" height="40" rx="4" />
      <text class="seg" :x="X0 + w(20.2) + w(2.02) / 2" y="30" text-anchor="middle">zh</text>
      <rect class="fake" :x="X0 + w(22.22)" y="4" :width="w(2.02) - 2" height="40" rx="4" />
      <text class="seg" :x="X0 + w(22.22) + w(2.02) / 2" y="30" text-anchor="middle">fake</text>
      <text class="total" :x="X0 + w(24.24) + 12" y="30">24.2B</text>
    </g>
  </svg>
  <svg v-else class="setup-data" viewBox="0 0 820 300" role="img">
    <!-- Multi-Train -->
    <g transform="translate(0, 40)">
      <text class="name" :x="X0 - 16" y="30" text-anchor="end">Multi-Train</text>
      <g v-for="(m, i) in multi" :key="m.k">
        <rect :class="m.cls" :x="X0 + offs(i)" y="4" :width="w(m.b) - 2" height="40" rx="4" />
        <text class="seg" :x="X0 + offs(i) + w(m.b) / 2" y="30" text-anchor="middle">{{ m.k }}</text>
      </g>
      <text class="total" :x="X0 + w(20.2) + 12" y="30">20.2B</text>
    </g>

    <!-- Mono-Train -->
    <g v-if="step >= 1" transform="translate(0, 120)">
      <text class="name" :x="X0 - 16" y="30" text-anchor="end">Mono-Train</text>
      <rect class="en" :x="X0" y="4" :width="w(20.2) - 2" height="40" rx="4" />
      <text class="seg" :x="X0 + w(20.2) / 2" y="30" text-anchor="middle">en</text>
      <text class="total" :x="X0 + w(20.2) + 12" y="30">20.2B</text>
    </g>

    <!-- Oracle-Train -->
    <g v-if="step >= 2" transform="translate(0, 200)">
      <text class="name" :x="X0 - 16" y="30" text-anchor="end">Oracle-Train</text>
      <g v-for="(m, i) in multi" :key="m.k">
        <rect :class="m.cls" :x="X0 + offs(i)" y="4" :width="w(m.b) - 2" height="40" rx="4" />
        <text class="seg" :x="X0 + offs(i) + w(m.b) / 2" y="30" text-anchor="middle">{{ m.k }}</text>
      </g>
      <text v-if="step < 3" class="note" :x="X0 + w(10.1)" y="70" text-anchor="middle">= 同一批 Multi-Train 文件（逐 shard 相同）</text>
      <g v-if="step >= 3">
        <rect class="zh" :x="X0 + w(20.2)" y="4" :width="w(2.02) - 2" height="40" rx="4" />
        <text class="seg" :x="X0 + w(20.2) + w(2.02) / 2" y="30" text-anchor="middle">zh</text>
        <rect class="fake" :x="X0 + w(22.22)" y="4" :width="w(2.02) - 2" height="40" rx="4" />
        <text class="seg" :x="X0 + w(22.22) + w(2.02) / 2" y="30" text-anchor="middle">fake</text>
        <text class="total" :x="X0 + w(24.24) + 12" y="30">24.2B</text>
        <text class="note add" :x="X0 + w(22.22)" y="70" text-anchor="middle">＋ 加上去，不是替換</text>
      </g>
    </g>
  </svg>
</template>

<style scoped>
.setup-data { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.name { fill: var(--google-ink); font-size: 19px; font-weight: 600; }
.seg { fill: white; font-size: 16px; font-weight: 600; }
.total { fill: var(--google-muted); font-size: 17px; }
.note { fill: var(--google-muted); font-size: 15px; }
.note.add { fill: var(--google-ink); font-weight: 600; }
.en { fill: var(--google-blue-dark); }
.es { fill: var(--google-blue); }
.de { fill: #7baaf7; }
.ru { fill: #a1c2fa; }
.zh { fill: var(--google-red); }
.fake { fill: var(--google-green); }
</style>
