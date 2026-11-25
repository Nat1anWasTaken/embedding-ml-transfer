<script setup lang="ts">
// CLWE in one picture: two monolingual skip-gram spaces trained separately
// (each on its own subword vocabulary), then VecMap's orthogonal W rotates
// the foreign space onto English, which stays fixed.
// `mapped` = false shows the two spaces apart; true moves zh onto en.
const props = withDefaults(defineProps<{
  mapped?: boolean
}>(), {
  mapped: false,
})

// Word positions relative to the cloud centre (shared shape, as if perfectly isomorphic).
const words = [
  { en: 'dog', zh: '狗', x: -84, y: -52 },
  { en: 'cat', zh: '貓', x: -52, y: -20 },
  { en: 'flower', zh: '花', x: 30, y: 64 },
  { en: 'tree', zh: '樹', x: 96, y: -18 },
]

const enC = { x: 120, y: 150 }
const zhC = { x: 420, y: 150 }
// Before mapping the zh space is the same shape, rotated 120°.
const cos = Math.cos((2 * Math.PI) / 3)
const sin = Math.sin((2 * Math.PI) / 3)

function enPos(w: typeof words[number]) {
  return { x: enC.x + w.x, y: enC.y + w.y }
}

function zhPos(w: typeof words[number]) {
  if (props.mapped)
    return { x: enC.x + w.x + 8, y: enC.y + w.y + 18 }
  return { x: zhC.x + w.x * cos - w.y * sin, y: zhC.y + w.x * sin + w.y * cos }
}
</script>

<template>
  <svg class="clwe-map" viewBox="0 0 540 300" role="img">
    <text class="space-label en" :x="enC.x" y="24" text-anchor="middle">en skip-gram</text>
    <text :class="['space-label', 'zh', { gone: props.mapped }]" :x="zhC.x" y="24" text-anchor="middle">zh skip-gram</text>

    <g :class="['arrow', { gone: props.mapped }]">
      <path d="M 340 276 Q 260 306 180 276" />
      <text x="260" y="268" text-anchor="middle">W</text>
    </g>

    <g v-for="w in words" :key="w.en" class="word en" :style="{ transform: `translate(${enPos(w).x}px, ${enPos(w).y}px)` }">
      <circle r="7" />
      <text x="12" y="-8">{{ w.en }}</text>
    </g>
    <g v-for="w in words" :key="w.zh" class="word zh" :style="{ transform: `translate(${zhPos(w).x}px, ${zhPos(w).y}px)` }">
      <circle r="7" />
      <text x="-12" y="22" text-anchor="end">{{ w.zh }}</text>
    </g>
  </svg>
</template>

<style scoped>
.clwe-map {
  width: 100%;
  overflow: visible;
  font-family: var(--deck-font-sans);
}
.space-label { font-size: 16px; font-weight: 600; transition: opacity 400ms ease; }
.space-label.en { fill: var(--google-blue); }
.space-label.zh { fill: #7e57c2; }
.gone { opacity: 0; }
.arrow { transition: opacity 400ms ease; }
.arrow path { fill: none; stroke: var(--google-muted); stroke-width: 2; stroke-dasharray: 6 5; }
.arrow text { fill: var(--google-ink); font-size: 20px; font-style: italic; font-weight: 600; }
.word { transition: transform 1100ms cubic-bezier(0.2, 0.8, 0.2, 1); }
.word text { font-size: 17px; font-weight: 500; }
.word.en circle { fill: var(--google-blue); }
.word.en text { fill: var(--google-blue-dark); }
.word.zh circle { fill: #7e57c2; opacity: 0.9; }
.word.zh text { fill: #7e57c2; }
</style>
