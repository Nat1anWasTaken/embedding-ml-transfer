<script setup lang="ts">
import { computed } from 'vue'

/**
 * Experiment badge for the Results section.
 *
 *   <ResultsExpTag exp="core" />                 small badge, pinned top-right
 *   <ResultsExpTag exp="a4a" full />             full badge with every config chip
 *   <ResultsExpTag :exp="['qwen', 'core']" />    comparison badge "A vs B"
 *   <ResultsExpTag map />                        all experiments as aligned rows
 *
 * Chips that differ from the core experiment are blue. Every badge carries
 * view-transition-name `exp-<key>`, so the map row → intro badge → corner
 * badge morph into each other.
 */

type Key = 'core' | 'a4a' | 'a4c' | 'mask' | 'mono' | 'center' | 'qwen' | 'untie'

interface Field { key: string, label: string, base: string }

const FIELDS: Field[] = [
  { key: 'e', label: 'E', base: '凍結' },
  { key: 'src', label: 'E 來源', base: 'Oracle' },
  { key: 'data', label: '訓練資料', base: 'Multi' },
  { key: 'softmax', label: 'softmax', base: '遮罩' },
  { key: 'scale', label: '規模', base: '300M' },
  { key: 'head', label: '輸出頭', base: '綁定' },
  { key: 'infer', label: '推論期', base: '原始' },
]

const EXPS: Record<Key, { name: string, diff: Record<string, string> }> = {
  core: { name: '核心實驗', diff: {} },
  a4a: { name: '全部可訓', diff: { e: '可訓', softmax: '完整' } },
  a4c: { name: '只動 L<sub>train</sub> 列', diff: { e: '只動 L<sub>train</sub>' } },
  mask: { name: '遮罩消融', diff: { softmax: '遮罩/完整', scale: '150M' } },
  mono: { name: '純英文主體', diff: { data: 'Mono' } },
  center: { name: '推論期置中', diff: { infer: '置中' } },
  qwen: { name: 'Qwen3 donor', diff: { src: 'Qwen3' } },
  untie: { name: '解綁輸出頭', diff: { head: '解綁' } },
}
const ORDER = Object.keys(EXPS) as Key[]

const props = withDefaults(defineProps<{
  exp?: Key | Key[]
  full?: boolean
  map?: boolean
  /** Pin the small badge to the slide's top-right corner. */
  corner?: boolean
}>(), { exp: 'core', full: false, map: false, corner: true })

const keys = computed(() => (Array.isArray(props.exp) ? props.exp : [props.exp]))
const main = computed(() => keys.value[0])

function chips(k: Key) {
  return FIELDS.map(f => ({
    ...f,
    value: EXPS[k].diff[f.key] ?? f.base,
    changed: f.key in EXPS[k].diff,
  }))
}
function diffChips(k: Key) {
  return chips(k).filter(c => c.changed)
}
</script>

<template>
  <!-- Overview: every experiment as one aligned row -->
  <div v-if="props.map" class="exp-map">
    <div class="exp-map-head">
      <span />
      <span v-for="f in FIELDS" :key="f.key">{{ f.label }}</span>
    </div>
    <div v-for="(k, i) in ORDER" :key="k" class="exp-map-row" :style="{ viewTransitionName: `exp-${k}` }">
      <span class="exp-name"><b class="exp-no">{{ i + 1 }}</b><span v-html="EXPS[k].name" /></span>
      <span v-for="c in chips(k)" :key="c.key" class="exp-chip" :class="{ changed: c.changed }" v-html="c.value" />
    </div>
  </div>

  <!-- Intro: name + every config chip -->
  <div v-else-if="props.full" class="exp-full" :style="{ viewTransitionName: `exp-${main}` }">
    <span class="exp-name"><b class="exp-no">{{ ORDER.indexOf(main) + 1 }}</b><span v-html="EXPS[main].name" /></span>
    <span v-for="c in chips(main)" :key="c.key" class="exp-chip labeled" :class="{ changed: c.changed }">
      <small>{{ c.label }}</small><span v-html="c.value" />
    </span>
  </div>

  <!-- Small badge: name + only what differs from the core experiment -->
  <div v-else class="exp-mini" :class="{ corner: props.corner }" :style="{ viewTransitionName: `exp-${main}` }">
    <template v-for="(k, i) in keys" :key="k">
      <span v-if="i > 0" class="exp-vs">vs</span>
      <span class="exp-name"><b class="exp-no">{{ ORDER.indexOf(k) + 1 }}</b><span v-html="EXPS[k].name" /></span>
      <template v-if="keys.length === 1">
        <span v-for="c in diffChips(k)" :key="c.key" class="exp-chip changed labeled">
          <small>{{ c.label }}</small><span v-html="c.value" />
        </span>
      </template>
    </template>
  </div>
</template>

<style scoped>
.exp-map,
.exp-full,
.exp-mini {
  font-family: var(--deck-font-sans);
  line-height: 1.2;
}

.exp-name {
  display: inline-flex;
  align-items: center;
  gap: 0.4em;
  color: var(--google-ink);
  font-weight: 700;
  white-space: nowrap;
}

.exp-no {
  display: inline-grid;
  place-items: center;
  width: 1.45em;
  height: 1.45em;
  border-radius: 50%;
  background: var(--google-blue);
  color: white;
  font-size: 0.72em;
  font-weight: 700;
}

.exp-chip {
  display: inline-flex;
  align-items: baseline;
  justify-content: center;
  gap: 0.35em;
  padding: 0.18em 0.6em;
  border: 1px solid var(--google-border);
  border-radius: 999px;
  background: var(--google-surface);
  color: var(--google-muted);
  font-weight: 500;
  white-space: nowrap;
}

.exp-chip small {
  color: var(--google-gray);
  font-size: 0.78em;
  font-weight: 500;
}

.exp-chip.changed {
  border-color: var(--google-blue);
  background: var(--google-blue-soft);
  color: var(--google-blue-dark);
  font-weight: 700;
}

.exp-chip.changed small {
  color: var(--google-blue-dark);
  opacity: 0.75;
}

.exp-chip :deep(sub) {
  font-size: 0.7em;
}

/* Small corner badge */
.exp-mini {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.22rem 0.6rem 0.22rem 0.3rem;
  border: 1px solid var(--google-border);
  border-radius: 999px;
  background: white;
  font-size: 13px;
}

.exp-mini.corner {
  position: absolute;
  z-index: 10;
  top: 0.7rem;
  right: 1.2rem;
}

.exp-mini .exp-chip {
  padding: 0.08em 0.5em;
  font-size: 12px;
}

.exp-vs {
  color: var(--google-gray);
  font-size: 12px;
  font-weight: 500;
}

/* Full intro badge */
.exp-full {
  display: inline-flex;
  flex-wrap: nowrap;
  align-items: center;
  justify-content: center;
  gap: 0.35rem;
  font-size: 14px;
}

.exp-full .exp-name {
  margin-right: 0.3rem;
  font-size: 18px;
}

/* Overview map */
.exp-map {
  display: grid;
  gap: 0.42rem;
  font-size: 15px;
}

.exp-map-head,
.exp-map-row {
  display: grid;
  grid-template-columns: 11.5rem repeat(7, minmax(0, 1fr));
  align-items: center;
  column-gap: 0.4rem;
}

.exp-map-head span {
  color: var(--google-muted);
  font-size: 13px;
  font-weight: 600;
  text-align: center;
}

.exp-map-row .exp-name {
  font-size: 17px;
}
</style>
