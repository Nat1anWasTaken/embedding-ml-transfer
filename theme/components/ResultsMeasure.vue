<script setup lang="ts">
/**
 * Says, in plain words, which measure a Results chart shows.
 *
 *   <ResultsMeasure kind="diff" />   two models compared directly: BPB difference
 *   <ResultsMeasure kind="tau" />    placed on the Floor→Upper ruler: τ
 *   <ResultsMeasure kind="align" />  how much better than the shuffled copy
 *   <ResultsMeasure legend />        the convention, all kinds side by side
 *
 * Optional `note` adds a short qualifier after the wording.
 */
type Kind = 'diff' | 'bpb' | 'tau' | 'align' | 'delta' | 'bli' | 'shuffle'

const MEASURES: Record<Kind, { head: string, body: string, glyph: string }> = {
  diff: { head: '兩個模型直接比', body: 'BPB 差（同一批句子，越負越好）', glyph: '−' },
  bpb: { head: '直接看', body: 'BPB（同一批句子，越低越好）', glyph: 'b' },
  tau: { head: '放到 Floor→Upper 的尺上', body: 'τ', glyph: 'τ' },
  align: { head: '比打亂版好多少', body: '對齊部分', glyph: '⇄' },
  delta: { head: '翻譯增益', body: '給對的來源句，譯文好多少（Δ）', glyph: 'Δ' },
  bli: { head: 'embedding 本身還對齊嗎', body: 'BLI P@1', glyph: '≈' },
  shuffle: { head: '比自己的打亂版差多少', body: 'BPB 差（同一批句子）', glyph: '⇄' },
}

const props = withDefaults(defineProps<{
  kind?: Kind
  note?: string
  legend?: boolean
  /** Kinds dimmed in the legend (not introduced yet). */
  later?: Kind[]
}>(), { kind: 'tau', legend: false, later: () => [] })

const LEGEND: Kind[] = ['tau', 'delta', 'align', 'bpb']
</script>

<template>
  <div v-if="props.legend" class="measure-legend">
    <span class="legend-title">量法</span>
    <span v-for="k in LEGEND" :key="k" class="measure" :class="[k, { later: props.later.includes(k) }]">
      <b class="glyph">{{ MEASURES[k].glyph }}</b>
      <span><b>{{ MEASURES[k].head }}</b>：{{ MEASURES[k].body }}</span>
    </span>
  </div>
  <div v-else class="measure" :class="props.kind">
    <b class="glyph">{{ MEASURES[props.kind].glyph }}</b>
    <span><b>{{ MEASURES[props.kind].head }}</b>：{{ MEASURES[props.kind].body }}<template v-if="props.note"> · {{ props.note }}</template></span>
  </div>
</template>

<style scoped>
.measure {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  color: var(--google-muted);
  font-family: var(--deck-font-sans);
  font-size: 16px;
  line-height: 1.3;
}

.measure b {
  color: var(--google-ink);
  font-weight: 600;
}

.glyph {
  display: inline-grid;
  flex: none;
  place-items: center;
  width: 1.6em;
  height: 1.6em;
  border: 1.5px solid var(--google-border);
  border-radius: 0.4em;
  color: var(--google-muted) !important;
  font-size: 0.9em;
  font-weight: 700 !important;
}

.measure.tau .glyph,
.measure.align .glyph {
  border-color: var(--google-blue);
  color: var(--google-blue-dark) !important;
}


.measure.shuffle .glyph {
  border-color: var(--google-blue);
  color: var(--google-blue-dark) !important;
}

.measure.align .glyph {
  background: var(--google-blue-soft);
}

.measure-legend {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.5rem 1.6rem;
  padding-top: 0.7rem;
  border-top: 1px solid var(--google-border);
}

.legend-title {
  color: var(--google-gray);
  font-size: 14px;
  font-weight: 700;
  letter-spacing: 0.08em;
}

.measure-legend .measure {
  font-size: 15px;
}

.measure.later {
  opacity: 0.4;
}
</style>
