<script setup lang="ts">
import { computed } from 'vue'

const props = withDefaults(defineProps<{
  /** Which route: A freezes E, B freezes the body, C freezes the body and generates E. */
  route: 'A' | 'B' | 'C'
  /** Scale factor for the whole glyph. */
  scale?: number
  /** Shared view-transition-name so the glyph morphs between slides. */
  vt?: string
}>(), {
  scale: 1,
})

type State = 'frozen' | 'trained' | 'generated'

const states = computed<{ emb: State, body: State }>(() => {
  if (props.route === 'A')
    return { emb: 'frozen', body: 'trained' }
  if (props.route === 'B')
    return { emb: 'trained', body: 'frozen' }
  return { emb: 'generated', body: 'frozen' }
})

const tag: Record<State, string> = { frozen: '凍結', trained: '訓練', generated: '生成' }
</script>

<template>
  <!-- A miniature tied model read left to right: (H_θ →) E → body → Eᵀ -->
  <div class="org" :style="{ '--org-s': props.scale, viewTransitionName: props.vt }">
    <div class="org-letter">
      {{ props.route }}
    </div>
    <div class="org-model">
      <template v-if="props.route === 'C'">
        <div class="org-part">
          <div class="org-box org-hyper trained">
            H<sub>θ</sub>
          </div>
          <span class="trained">訓練</span>
        </div>
        <div class="org-arrow gen">
          →
        </div>
      </template>
      <div class="org-part">
        <div class="org-box org-emb" :class="states.emb">
          E
        </div>
        <span :class="states.emb">{{ tag[states.emb] }}</span>
      </div>
      <div class="org-arrow">
        →
      </div>
      <div class="org-part">
        <div class="org-body" :class="states.body">
          <i v-for="i in 5" :key="i" />
        </div>
        <span :class="states.body">主體{{ tag[states.body] }}</span>
      </div>
      <div class="org-arrow">
        →
      </div>
      <div class="org-part">
        <div class="org-box org-emb" :class="states.emb">
          Eᵀ
        </div>
        <span :class="states.emb">{{ tag[states.emb] }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.org {
  display: inline-flex;
  align-items: center;
  gap: calc(0.9rem * var(--org-s));
  font-size: calc(16px * var(--org-s));
}

.org-letter {
  color: var(--google-blue-dark);
  font-size: calc(44px * var(--org-s));
  font-weight: 700;
  line-height: 1;
}

.org-model {
  display: flex;
  align-items: flex-start;
  gap: 0.3em;
}

.org-part {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.35em;
}

.org-part span {
  font-size: 0.78em;
  font-weight: 600;
  white-space: nowrap;
}

.org-arrow {
  height: 3.2em;
  color: var(--google-gray);
  font-weight: 700;
  line-height: 3.2em;
}

.org-arrow.gen {
  color: var(--google-green-dark);
}

.org-box {
  display: flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
  width: 2.6em;
  height: 3.2em;
  border: 2px solid;
  border-radius: 0.4em;
  font-weight: 700;
}

.org-hyper {
  width: 2.8em;
}

.org-body {
  display: flex;
  gap: 0.22em;
  height: 3.2em;
}

.org-body i {
  box-sizing: border-box;
  width: 0.8em;
  height: 100%;
  border: 2px solid;
  border-radius: 0.25em;
}

/* Box colours by state; the label under each part takes the matching text colour */
.org-box.frozen,
.org-body.frozen i {
  border-style: dashed;
  border-color: var(--google-gray);
  background: var(--google-surface);
  color: var(--google-muted);
}

.org-box.trained,
.org-body.trained i {
  border-color: var(--google-blue);
  background: var(--google-blue-soft);
  color: var(--google-blue-dark);
}

.org-box.generated {
  border-color: var(--google-green);
  background: #e6f4ea;
  color: var(--google-green-dark);
}

span.frozen { color: var(--google-muted); }
span.trained { color: var(--google-blue-dark); }
span.generated { color: var(--google-green-dark); }
</style>
