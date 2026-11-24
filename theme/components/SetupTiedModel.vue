<script setup lang="ts">
/**
 * Tied decoder used in every Route-A run: E (lookup) → 24-layer body → Eᵀ (LM head).
 * step 1: the two ends are the same matrix; step 2: E is frozen, only the body trains.
 */
withDefaults(defineProps<{ step?: number }>(), { step: 99 })
</script>

<template>
  <svg class="setup-tied" viewBox="0 0 820 300" role="img">
    <defs>
      <marker id="tm-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
        <path d="M0,0 L10,5 L0,10 z" fill="#9aa0a6" />
      </marker>
    </defs>

    <!-- token in -->
    <text x="40" y="168" class="io" text-anchor="middle">token</text>
    <line x1="66" y1="162" x2="104" y2="162" class="flow" marker-end="url(#tm-arrow)" />

    <!-- input E -->
    <g :class="['emb', { frozen: step >= 2 }]">
      <rect x="110" y="102" width="92" height="120" rx="8" />
      <text x="156" y="168" text-anchor="middle" class="mat">E</text>
      <text x="156" y="246" text-anchor="middle" class="sub">查第 x 列</text>
    </g>
    <line x1="208" y1="162" x2="262" y2="162" class="flow" marker-end="url(#tm-arrow)" />

    <!-- body -->
    <g class="body">
      <rect v-for="i in 8" :key="i" :x="268 + (i - 1) * 36" y="96" width="28" height="132" rx="5" />
      <text x="408" y="252" text-anchor="middle" class="sub">Transformer 主體 · L = 24 · d = 1024</text>
      <text v-if="step >= 2" x="408" y="80" text-anchor="middle" class="train-tag">只訓練這裡</text>
    </g>
    <line x1="558" y1="162" x2="612" y2="162" class="flow" marker-end="url(#tm-arrow)" />

    <!-- output Eᵀ -->
    <g :class="['emb', { frozen: step >= 2 }]">
      <rect x="618" y="102" width="92" height="120" rx="8" />
      <text x="664" y="168" text-anchor="middle" class="mat">Eᵀ</text>
      <text x="664" y="246" text-anchor="middle" class="sub">softmax(E h)</text>
    </g>
    <line x1="716" y1="162" x2="754" y2="162" class="flow" marker-end="url(#tm-arrow)" />
    <text x="786" y="168" class="io" text-anchor="middle">p(y)</text>

    <!-- tie arc -->
    <g v-if="step >= 1" class="tie">
      <path d="M156,96 C156,30 664,30 664,96" />
      <text x="410" y="34" text-anchor="middle">同一個矩陣（tied）</text>
    </g>

    <!-- frozen badges -->
    <g v-if="step >= 2" class="lock">
      <text x="156" y="282" text-anchor="middle">凍結</text>
      <text x="664" y="282" text-anchor="middle">凍結</text>
    </g>
  </svg>
</template>

<style scoped>
.setup-tied { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.flow { stroke: var(--google-gray); stroke-width: 2.5; }
.io { fill: var(--google-muted); font-size: 18px; }
.emb rect { fill: var(--google-blue-soft); stroke: var(--google-blue); stroke-width: 2.5; transition: all 400ms ease; }
.emb.frozen rect { fill: #eef1f4; stroke: var(--google-gray); stroke-dasharray: 6 4; }
.mat { fill: var(--google-blue-dark); font-size: 34px; font-weight: 700; }
.emb.frozen .mat { fill: var(--google-muted); }
.sub { fill: var(--google-muted); font-size: 15px; }
.body rect { fill: var(--google-surface); stroke: var(--google-border); stroke-width: 2; }
.train-tag { fill: var(--google-green-dark); font-size: 16px; font-weight: 600; }
.tie path { fill: none; stroke: var(--google-blue); stroke-width: 2.5; stroke-dasharray: 7 5; }
.tie text { fill: var(--google-blue-dark); font-size: 17px; font-weight: 600; }
.lock text { fill: var(--google-muted); font-size: 15px; font-weight: 600; }
</style>
