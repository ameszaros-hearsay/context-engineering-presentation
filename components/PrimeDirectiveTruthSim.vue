<script setup lang="ts">
import { computed, ref } from 'vue'
import ContextWindowCompositionBar from './ContextWindowCompositionBar.vue'

type RectKind = 'relevant' | 'found'

type Rect = {
  id: string
  kind: RectKind
  x: number
  y: number
  w: number
  h: number
}

const mode = ref<'truthful' | 'hidden'>('truthful')

const truthfulRelevantBlocks: Rect[] = [
  { id: 'rel-1', kind: 'relevant', x: 12, y: 14, w: 22, h: 18 },
  { id: 'rel-2', kind: 'relevant', x: 44, y: 16, w: 22, h: 18 },
  { id: 'rel-3', kind: 'relevant', x: 18, y: 62, w: 24, h: 18 },
  { id: 'rel-4', kind: 'relevant', x: 62, y: 60, w: 20, h: 18 },
]

const hiddenRelevantBlocks: Rect[] = [
  { id: 'rel-1', kind: 'relevant', x: 10, y: 14, w: 26, h: 20 },
  { id: 'rel-2', kind: 'relevant', x: 42, y: 16, w: 26, h: 20 },
  { id: 'rel-3', kind: 'relevant', x: 16, y: 60, w: 28, h: 20 },
  { id: 'rel-4', kind: 'relevant', x: 60, y: 58, w: 24, h: 20 },
]

const truthfulFound: Rect[] = [
  { id: 'f-1', kind: 'found', x: 10, y: 12, w: 28, h: 22 },
  { id: 'f-2', kind: 'found', x: 42, y: 14, w: 28, h: 22 },
  { id: 'f-3', kind: 'found', x: 16, y: 58, w: 30, h: 22 },
  { id: 'f-4', kind: 'found', x: 60, y: 58, w: 26, h: 22 },
]

const hiddenFound: Rect[] = [
  // Misses two relevant pockets
  { id: 'f-1', kind: 'found', x: 6, y: 10, w: 42, h: 26 },
  { id: 'f-2', kind: 'found', x: 48, y: 12, w: 46, h: 26 },
]

const relevantBlocks = computed(() => (mode.value === 'hidden' ? hiddenRelevantBlocks : truthfulRelevantBlocks))

const scenario = computed(() => {
  if (mode.value === 'hidden') {
    return {
      title: 'Hidden exceptions',
      subtitle: 'The repo lies: relevant units are harder to find.',
      found: hiddenFound,
      usage: 82,
      segments: [
        { kind: 'overhead' as const, value: 18 },
        { kind: 'signal' as const, value: 16 },
        { kind: 'noise' as const, value: 48 },
      ],
      caption: 'Bigger fill, polluted context',
    }
  }

  return {
    title: 'Truthful repo',
    subtitle: 'Structure and names match reality.',
    found: truthfulFound,
    usage: 46,
    segments: [
      { kind: 'overhead' as const, value: 18 },
      { kind: 'signal' as const, value: 22 },
      { kind: 'noise' as const, value: 6 },
    ],
    caption: 'Smaller fill, cleaner signal',
  }
})
</script>

<template>
  <section class="sim" aria-label="Prime directive good vs bad">
    <div class="tabs" role="tablist" aria-label="Scenario">
      <button class="tab" :class="{ active: mode === 'truthful' }" type="button" @click="mode = 'truthful'">
        Truthful repo
      </button>
      <button class="tab" :class="{ active: mode === 'hidden' }" type="button" @click="mode = 'hidden'">
        Hidden exceptions
      </button>
    </div>

    <div class="panel">
      <header class="panel-head">
        <h3>{{ scenario.title }}</h3>
        <p>{{ scenario.subtitle }}</p>
      </header>

      <div class="repo">
        <div
          v-for="block in relevantBlocks"
          :key="block.id"
          class="rect relevant"
          :style="{ left: `${block.x}%`, top: `${block.y}%`, width: `${block.w}%`, height: `${block.h}%` }"
        />

        <div
          v-for="block in scenario.found"
          :key="block.id"
          class="rect found"
          :style="{ left: `${block.x}%`, top: `${block.y}%`, width: `${block.w}%`, height: `${block.h}%` }"
        />
      </div>

      <div class="cw">
        <ContextWindowCompositionBar
          :usage="scenario.usage"
          :segments="scenario.segments"
          :caption="scenario.caption"
          collapse-overhead
        />
      </div>
    </div>

    <div class="legend" aria-label="Legend">
      <span class="legend-item"><i class="chip relevant" /> Relevant files</span>
      <span class="legend-item"><i class="chip found" /> Found references</span>
    </div>
  </section>
</template>

<style scoped>
.sim {
  display: grid;
  gap: 0.3rem;
}

.tabs {
  display: flex;
  gap: 0.36rem;
  flex-wrap: wrap;
}

.tab {
  border-radius: 999px;
  border: 2px solid #95a9be;
  background: #e9f0f6;
  color: #1f2937;
  padding: 0.12rem 0.4rem;
  font-size: 0.56rem;
  font-weight: 800;
  cursor: pointer;
}

.tab.active {
  border-color: #8bae74;
  background: #deecd0;
}

.panel {
  border: 1px solid #8ea3b8;
  border-radius: 10px;
  background: #f8fbff;
  padding: 0.28rem 0.35rem;
}

.panel-head h3 {
  margin: 0;
  font-size: 0.7rem;
  font-weight: 800;
  color: #1f2937;
}

.panel-head p {
  margin: 0.06rem 0 0.18rem;
  font-size: 0.56rem;
  color: #475569;
}

.repo {
  position: relative;
  aspect-ratio: 2.8 / 1;
  border-radius: 9px;
  background:
    linear-gradient(#dce8f2 1px, transparent 1px),
    linear-gradient(90deg, #dce8f2 1px, transparent 1px);
  background-size: 12px 12px, 12px 12px;
  background-color: #eef5fb;
  border: 1px solid #b5c6d5;
  overflow: hidden;
}

.rect {
  position: absolute;
  border-radius: 6px;
}

.relevant {
  background: rgba(22, 163, 74, 0.24);
  border: 2px solid rgba(22, 163, 74, 0.72);
}

.found {
  background: rgba(37, 99, 235, 0.2);
  border: 2px solid rgba(30, 64, 175, 0.68);
}

.cw {
  margin-top: 0.22rem;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.45rem;
  align-items: center;
  font-size: 0.52rem;
  color: #dce7f5;
  font-weight: 700;
  text-shadow: 0 1px 0 rgba(15, 23, 42, 0.35);
}

.legend-item {
  display: inline-flex;
  align-items: center;
  gap: 0.28rem;
}

.chip {
  width: 0.6rem;
  height: 0.6rem;
  border-radius: 4px;
  display: inline-block;
  box-shadow: 0 0 0 1px rgba(15, 23, 42, 0.45);
}

.chip.relevant {
  background: rgba(22, 163, 74, 0.24);
  border: 2px solid rgba(22, 163, 74, 0.72);
}

.chip.found {
  background: rgba(37, 99, 235, 0.2);
  border: 2px solid rgba(30, 64, 175, 0.68);
}
</style>
