<script setup lang="ts">
import { computed, ref } from 'vue'
import ContextWindowCompositionBar from './ContextWindowCompositionBar.vue'

type Rect = {
  id: string
  x: number
  y: number
  w: number
  h: number
}

const mode = ref<'blind' | 'anchored'>('blind')

const relevantBlocks: Rect[] = [
  { id: 'r-1', x: 12, y: 12, w: 30, h: 16 },
  { id: 'r-2', x: 58, y: 18, w: 28, h: 18 },
  { id: 'r-3', x: 18, y: 62, w: 26, h: 18 },
  { id: 'r-4', x: 62, y: 62, w: 22, h: 16 },
]

const blindBlocks: Rect[] = [
  { id: 'n-1', x: 6, y: 6, w: 60, h: 30 },
  { id: 'n-2', x: 36, y: 34, w: 56, h: 30 },
  { id: 'n-3', x: 8, y: 58, w: 42, h: 30 },
]

const anchoredBlocks: Rect[] = [
  { id: 'f-1', x: 10, y: 10, w: 36, h: 20 },
  { id: 'f-2', x: 56, y: 16, w: 34, h: 22 },
  { id: 'f-3', x: 16, y: 60, w: 30, h: 22 },
  { id: 'f-4', x: 60, y: 60, w: 28, h: 20 },
]

const scenario = computed(() => {
  if (mode.value === 'blind') {
    return {
      title: 'Blind discovery',
      subtitle: 'Reads wide surface and misses relevant pockets',
      usage: 78,
      segments: [
        { kind: 'overhead' as const, value: 18 },
        { kind: 'signal' as const, value: 20 },
        { kind: 'noise' as const, value: 40 },
      ],
      caption: 'More fill, more noise',
    }
  }

  return {
    title: 'Anchored discovery',
    subtitle: 'Uses exact file/symbol references',
    usage: 44,
    segments: [
      { kind: 'overhead' as const, value: 18 },
      { kind: 'signal' as const, value: 20 },
      { kind: 'noise' as const, value: 6 },
    ],
    caption: 'More headroom, cleaner signal',
  }
})

const readBlocks = computed(() => (mode.value === 'blind' ? blindBlocks : anchoredBlocks))
</script>

<template>
  <section class="coverage" aria-label="Relevant coverage comparison">
    <div class="tabs" role="tablist" aria-label="Discovery mode">
      <button class="tab" :class="{ active: mode === 'blind' }" type="button" @click="mode = 'blind'">
        Blind discovery
      </button>
      <button class="tab" :class="{ active: mode === 'anchored' }" type="button" @click="mode = 'anchored'">
        Anchored discovery
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
          :key="`rel-${block.id}`"
          class="rect relevant"
          :style="{ left: `${block.x}%`, top: `${block.y}%`, width: `${block.w}%`, height: `${block.h}%` }"
        />

        <div
          v-for="block in readBlocks"
          :key="`read-${mode}-${block.id}`"
          class="rect read"
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

    <div class="legend">
      <span class="legend-item"><i class="chip relevant" /> Target relevant code</span>
      <span class="legend-item"><i class="chip read" /> Read area</span>
    </div>
  </section>
</template>

<style scoped>
.coverage {
  margin-top: 0.1rem;
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
  background: rgba(22, 163, 74, 0.26);
  border: 2px solid rgba(22, 163, 74, 0.75);
}

.read {
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
  background: rgba(22, 163, 74, 0.26);
  border: 2px solid rgba(22, 163, 74, 0.75);
}

.chip.read {
  background: rgba(37, 99, 235, 0.2);
  border: 2px solid rgba(30, 64, 175, 0.68);
}

@media (max-width: 980px) {
  .tabs {
    justify-content: center;
  }
}
</style>
