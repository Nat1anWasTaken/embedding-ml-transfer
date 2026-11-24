<script setup lang="ts">
// The tied matrix E drawn as language bands of rows.
// state: `all` = every language present; `banned` = L_ban rows absent during
// training (dashed, empty); `returned` = L_ban rows put back at inference.
const props = withDefaults(defineProps<{
  state?: 'all' | 'banned' | 'returned'
}>(), {
  state: 'all',
})

const train = [
  { code: 'en', color: '#4285f4', h: 64 },
  { code: 'es', color: '#34a853', h: 36 },
  { code: 'de', color: '#fbbc04', h: 36 },
  { code: 'ru', color: '#ea4335', h: 36 },
]
</script>

<template>
  <div class="intro-rows" style="view-transition-name: intro-rows">
    <div class="matrix">
      <div v-for="b in train" :key="b.code" class="band" :style="{ height: `${b.h}px`, background: b.color }">
        <span>{{ b.code }}</span>
      </div>
      <div :class="['band', 'ban', props.state]" style="height: 64px">
        <span>zh</span>
      </div>
    </div>
    <div class="side">
      <div class="side-train">L<sub>train</sub>：主體訓練時看得到</div>
      <div :class="['side-ban', props.state]">
        <template v-if="props.state === 'all'">
          L<sub>ban</sub>：另一組列
        </template>
        <template v-else-if="props.state === 'banned'">
          L<sub>ban</sub>：訓練時主體從沒看過
        </template>
        <template v-else>
          L<sub>ban</sub>：推論時放回
        </template>
      </div>
    </div>
    <div v-if="props.state !== 'all'" class="frozen">E 凍結 · d = 1024</div>
  </div>
</template>

<style scoped>
.intro-rows {
  position: relative;
  display: grid;
  grid-template-columns: 300px auto;
  column-gap: 1.6rem;
  align-items: stretch;
  margin-top: 0.6rem;
  text-align: left;
}
.matrix {
  display: flex;
  flex-direction: column;
  border: 2px solid var(--google-ink);
  border-radius: 6px;
  overflow: hidden;
}
.band {
  display: flex;
  align-items: center;
  padding-left: 0.8rem;
  color: white;
  font-size: 17px;
  font-weight: 600;
  opacity: 0.88;
  transition: background 500ms ease, opacity 500ms ease;
}
.band.ban { background: #7e57c2; }
.band.ban.banned {
  background: repeating-linear-gradient(135deg, white 0 8px, #ede7f6 8px 16px);
  color: #7e57c2;
  outline: 2px dashed #7e57c2;
  outline-offset: -4px;
}
.band.ban.returned { background: #7e57c2; opacity: 1; box-shadow: inset 0 0 0 3px #4a148c; }
.side {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  font-size: 20px;
}
.side-train { margin-top: 3.2rem; color: var(--google-muted); }
.side-ban { margin-bottom: 1.1rem; color: #7e57c2; font-weight: 600; }
.frozen {
  position: absolute;
  top: -1.8rem;
  left: 0;
  color: var(--google-muted);
  font-size: 15px;
}
</style>
