<script setup lang="ts">
import { ref } from 'vue'
import { onSlideEnter, onSlideLeave } from '@slidev/client'
import { resolveAssetUrl } from '../layoutHelper'

const props = withDefaults(defineProps<{
  /** Page images, front page first, e.g. ['/papers/artetxe2020/p1.png', ...]. */
  pages: string[]
  /** Short citation shown under the stack, e.g. "Artetxe et al." */
  title?: string
  /** Venue and year, e.g. "ACL 2020" */
  venue?: string
  /** Width of the front page, any CSS length. */
  width?: string
  /** `fan` rotates the back pages, `cascade` offsets them diagonally. */
  spread?: 'fan' | 'cascade'
  /** Shared view-transition-name so the stack morphs between slides. */
  vt?: string
}>(), {
  width: '15rem',
  spread: 'fan',
})

// The stack starts collapsed and spreads each time its slide becomes active.
const open = ref(false)
onSlideEnter(() => {
  open.value = false
  requestAnimationFrame(() => requestAnimationFrame(() => (open.value = true)))
})
onSlideLeave(() => (open.value = false))

function pageStyle(i: number) {
  // i = 0 is the front page; larger i sits further back in the stack.
  const back = i
  const fan = props.spread === 'fan'
  const transform = open.value
    ? fan
      ? `translate(${back * 1.6}rem, ${back * 0.35}rem) rotate(${back * 4}deg)`
      : `translate(${back * 1.4}rem, ${back * -1.1}rem)`
    : 'translate(0, 0.6rem) rotate(0deg)'
  return {
    zIndex: props.pages.length - i,
    transform,
    opacity: open.value ? 1 : 0,
    transitionDelay: `${(props.pages.length - 1 - i) * 90}ms`,
  }
}
</script>

<template>
  <figure class="paper-stack" :class="spread" :style="{ '--paper-w': props.width, viewTransitionName: props.vt }">
    <div class="paper-stack-pages">
      <img
        v-for="(src, i) in props.pages"
        :key="src"
        :src="resolveAssetUrl(src)"
        class="paper-stack-page"
        :style="pageStyle(i)"
        alt=""
      >
    </div>
    <figcaption v-if="props.title || props.venue">
      <b v-if="props.title">{{ props.title }}</b>
      <span v-if="props.venue">{{ props.venue }}</span>
    </figcaption>
  </figure>
</template>

<style scoped>
.paper-stack {
  display: inline-flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 1.1rem;
  margin: 0;
}

.paper-stack-pages {
  position: relative;
  width: calc(var(--paper-w) + 3.5rem);
  aspect-ratio: 8.5 / 11;
  max-width: 100%;
}

.paper-stack-page {
  position: absolute;
  top: 0;
  left: 0;
  width: var(--paper-w);
  border-radius: 3px;
  background: white;
  box-shadow: 0 1px 2px rgba(60, 64, 67, 0.18), 0 10px 28px rgba(60, 64, 67, 0.16);
  transform-origin: 30% 90%;
  transition:
    transform 700ms cubic-bezier(0.2, 0.8, 0.2, 1),
    opacity 400ms ease;
}

figcaption {
  display: flex;
  flex-direction: column;
  line-height: 1.25;
}

figcaption b {
  font-size: 18px;
  font-weight: 600;
}

figcaption span {
  color: var(--google-muted);
  font-size: 15px;
}
</style>
