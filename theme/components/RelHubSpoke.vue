<script setup lang="ts">
// Hub-and-spoke sketch for the Semantic Hub slide.
// step >= 1: the 中文 spoke is detached (dashed) = "swap one spoke".
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

const spokes = [
  { x: 210, y: 46, label: 'English' },
  { x: 360, y: 120, label: '程式碼' },
  { x: 320, y: 290, label: '算式' },
  { x: 100, y: 290, label: '影音' },
]
</script>

<template>
  <svg class="hub" viewBox="0 0 420 340" role="img">
    <line v-for="s in spokes" :key="`l${s.label}`" x1="210" y1="170" :x2="s.x" :y2="s.y" class="spoke" />
    <line x1="210" y1="170" x2="62" y2="120" :class="['spoke', 'zh', { off: props.step >= 1 }]" />
    <circle cx="210" cy="170" r="60" class="hub-c" />
    <text x="210" y="166" text-anchor="middle" class="hub-t">hub</text>
    <text x="210" y="192" text-anchor="middle" class="hub-s">語意</text>
    <g v-for="s in spokes" :key="s.label">
      <circle :cx="s.x" :cy="s.y" r="36" class="node" />
      <text :x="s.x" :y="s.y + 6" text-anchor="middle" class="node-t">{{ s.label }}</text>
    </g>
    <g :class="['zh-node', { off: props.step >= 1 }]">
      <circle cx="62" cy="120" r="36" class="node" />
      <text x="62" y="126" text-anchor="middle" class="node-t">中文</text>
    </g>
  </svg>
</template>

<style scoped>
.hub { width: 100%; overflow: visible; font-family: var(--deck-font-sans); }
.spoke { stroke: var(--google-gray); stroke-width: 2.5; transition: stroke 400ms; }
.spoke.zh.off { stroke: #7e57c2; stroke-dasharray: 7 6; }
.hub-c { fill: var(--google-blue); }
.hub-t { fill: white; font-size: 22px; font-weight: 600; }
.hub-s { fill: white; font-size: 15px; }
.node { fill: var(--google-blue-soft); stroke: none; transition: fill 400ms, stroke 400ms; }
.node-t { fill: var(--google-ink); font-size: 16px; font-weight: 500; }
.zh-node { transition: transform 600ms cubic-bezier(0.2, 0.8, 0.2, 1); }
.zh-node.off { transform: translate(-14px, -10px); }
.zh-node.off .node { fill: #ede7f6; stroke: #7e57c2; stroke-width: 2.5; stroke-dasharray: 5 4; }
.zh-node.off .node-t { fill: #7e57c2; font-weight: 600; }
</style>
