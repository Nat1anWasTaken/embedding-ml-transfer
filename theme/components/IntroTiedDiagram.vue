<script setup lang="ts">
// Schematic of a decoder LM: input embedding -> Transformer body -> output head.
// `latent` mode shows a logit-lens style "concept" in the middle layers;
// `tied` mode shows that the input and output matrices are the same E.
const props = withDefaults(defineProps<{
  mode?: 'latent' | 'tied'
  /** Pass `$clicks` from the slide. */
  step?: number
  input?: string
  output?: string
  latent?: string
}>(), {
  mode: 'latent',
  step: 0,
  input: 'fleur',
  output: '花',
  latent: 'flower',
})

const bands = ['#4285f4', '#34a853', '#fbbc04', '#ea4335', '#9aa0a6', '#8ab4f8']
const layers = [0, 1, 2, 3, 4, 5, 6]
const layerX = (i: number) => 268 + i * 44
</script>

<template>
  <svg class="intro-tied" viewBox="0 0 820 300" role="img">
    <defs>
      <marker id="intro-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
        <path d="M0,0 L10,5 L0,10 z" fill="#9aa0a6" />
      </marker>
    </defs>

    <!-- input token -->
    <rect x="20" y="135" width="92" height="44" rx="10" class="tok" />
    <text x="66" y="164" text-anchor="middle" class="tok-text">{{ props.input }}</text>
    <line x1="114" y1="157" x2="146" y2="157" class="flow" marker-end="url(#intro-arrow)" />

    <!-- input embedding E -->
    <g :class="['mat', { hot: props.mode === 'tied' && props.step >= 1 }]">
      <rect v-for="(c, i) in bands" :key="`l${i}`" x="150" :y="82 + i * 25" width="70" height="25" :fill="c" opacity="0.8" />
      <rect x="150" y="82" width="70" height="150" class="mat-frame" fill="none" />
    </g>
    <text x="185" y="70" text-anchor="middle" class="mat-label">E</text>
    <text x="185" y="258" text-anchor="middle" class="cap">讀入</text>

    <line x1="224" y1="157" x2="258" y2="157" class="flow" marker-end="url(#intro-arrow)" />

    <!-- body -->
    <rect
      v-for="i in layers"
      :key="`b${i}`"
      :x="layerX(i)"
      y="97"
      width="34"
      height="120"
      rx="7"
      :class="['layer', { concept: props.mode === 'latent' && props.step >= 1 && i >= 2 && i <= 4 }]"
    />
    <text x="422" y="245" text-anchor="middle" class="cap">Transformer 主體</text>

    <g :class="['latent', { on: props.mode === 'latent' && props.step >= 1 }]">
      <line x1="422" y1="62" x2="422" y2="95" class="latent-line" />
      <rect x="362" y="26" width="120" height="38" rx="19" class="latent-pill" />
      <text x="422" y="51" text-anchor="middle" class="latent-text">{{ props.latent }}</text>
    </g>

    <line x1="580" y1="157" x2="614" y2="157" class="flow" marker-end="url(#intro-arrow)" />

    <!-- output head -->
    <g :class="['mat', { hot: props.mode === 'tied' && props.step >= 1 }]">
      <rect v-for="(c, i) in bands" :key="`r${i}`" x="618" :y="82 + i * 25" width="70" height="25" :fill="props.mode === 'tied' && props.step >= 1 ? c : '#dadce0'" :opacity="props.mode === 'tied' && props.step >= 1 ? 0.8 : 1" class="band" />
      <rect x="618" y="82" width="70" height="150" class="mat-frame" fill="none" />
    </g>
    <text x="653" y="70" text-anchor="middle" class="mat-label">{{ props.mode === 'tied' && props.step >= 1 ? 'Eᵀ' : 'LM head' }}</text>
    <text x="653" y="258" text-anchor="middle" class="cap">寫出</text>

    <line x1="692" y1="157" x2="720" y2="157" class="flow" marker-end="url(#intro-arrow)" />
    <rect x="724" y="135" width="76" height="44" rx="10" class="tok" />
    <text x="762" y="164" text-anchor="middle" class="tok-text">{{ props.output }}</text>

    <!-- tie arc -->
    <g :class="['tie', { on: props.mode === 'tied' && props.step >= 1 }]">
      <path d="M185,60 C185,0 653,0 653,60" class="tie-path" />
      <rect x="349" y="2" width="146" height="30" rx="15" class="tie-pill" />
      <text x="422" y="23" text-anchor="middle" class="tie-text">同一個矩陣</text>
    </g>
  </svg>
</template>

<style scoped>
.intro-tied { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.tok { fill: white; stroke: var(--google-border); stroke-width: 1.5; }
.tok-text { fill: var(--google-ink); font-size: 20px; font-weight: 500; }
.flow { stroke: var(--google-gray); stroke-width: 2; }
.mat-frame { stroke: var(--google-ink); stroke-width: 1.5; rx: 4; transition: stroke 400ms, stroke-width 400ms; }
.mat.hot .mat-frame { stroke: var(--google-blue-dark); stroke-width: 4; }
.band { transition: fill 500ms ease, opacity 500ms ease; }
.mat-label { fill: var(--google-ink); font-size: 20px; font-weight: 600; }
.cap { fill: var(--google-muted); font-size: 16px; }
.layer { fill: var(--google-blue-soft); stroke: var(--google-blue); stroke-width: 1.5; transition: fill 500ms ease; }
.layer.concept { fill: #c6dafc; }
.latent { opacity: 0; transform: translateY(8px); transition: opacity 400ms ease, transform 500ms ease; }
.latent.on { opacity: 1; transform: none; }
.latent-line { stroke: var(--google-blue); stroke-width: 2; stroke-dasharray: 4 4; }
.latent-pill { fill: var(--google-blue); }
.latent-text { fill: white; font-size: 19px; font-weight: 600; }
.tie { opacity: 0; transition: opacity 500ms ease; }
.tie.on { opacity: 1; }
.tie-path { fill: none; stroke: var(--google-blue-dark); stroke-width: 2.5; stroke-dasharray: 6 5; }
.tie-pill { fill: var(--google-blue-dark); }
.tie-text { fill: white; font-size: 16px; font-weight: 600; }
</style>
