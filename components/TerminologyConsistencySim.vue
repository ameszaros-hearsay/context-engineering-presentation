<script setup lang="ts">
import { computed, ref } from 'vue'
import ContextWindowCompositionBar from './ContextWindowCompositionBar.vue'

type RectKind = 'relevant' | 'found' | 'ambiguous'

type Rect = {
  id: string
  kind: RectKind
  x: number
  y: number
  w: number
  h: number
}

const mode = ref<'consistent' | 'overloaded'>('consistent')

const relevantZones: Rect[] = [
  { id: 'rel-1', kind: 'relevant', x: 10, y: 12, w: 30, h: 22 },
  { id: 'rel-2', kind: 'relevant', x: 54, y: 14, w: 30, h: 22 },
  { id: 'rel-3', kind: 'relevant', x: 18, y: 60, w: 30, h: 22 },
]

const ambiguousZones: Rect[] = [
  // Kept disjoint from relevant zones to keep the set visualization unambiguous.
  // In the overloaded case these are (mostly) a subset of what the agent "finds".
  { id: 'amb-1', kind: 'ambiguous', x: 84, y: 10, w: 10, h: 26 },
  { id: 'amb-2', kind: 'ambiguous', x: 50, y: 56, w: 12, h: 32 },
]

const consistentFound: Rect[] = [
  { id: 'f-1', kind: 'found', x: 8, y: 10, w: 34, h: 26 },
  { id: 'f-2', kind: 'found', x: 52, y: 12, w: 34, h: 26 },
  { id: 'f-3', kind: 'found', x: 16, y: 58, w: 34, h: 26 },
]

const overloadedFound: Rect[] = [
  { id: 'f-1', kind: 'found', x: 6, y: 6, w: 40, h: 34 },
  { id: 'f-2', kind: 'found', x: 52, y: 6, w: 42, h: 34 },
  { id: 'f-3', kind: 'found', x: 10, y: 54, w: 54, h: 36 },
]

const scenario = computed(() => {
  if (mode.value === 'overloaded') {
    return {
      title: 'Overloaded terms',
      subtitle: 'Same word matches multiple unrelated concepts.',
      showAmbiguous: true,
      found: overloadedFound,
      usage: 80,
      segments: [
        { kind: 'overhead' as const, value: 18 },
        { kind: 'signal' as const, value: 18 },
        { kind: 'noise' as const, value: 44 },
      ],
      caption: 'More matches, more pollution',
    }
  }

  return {
    title: 'Consistent vocabulary',
    subtitle: 'Terms map cleanly to one concept.',
    showAmbiguous: false,
    found: consistentFound,
    usage: 48,
    segments: [
      { kind: 'overhead' as const, value: 18 },
      { kind: 'signal' as const, value: 24 },
      { kind: 'noise' as const, value: 6 },
    ],
    caption: 'Cleaner retrieval, more headroom',
  }
})
</script>

<template>
  <section class="sim" aria-label="Terminology consistency good vs bad">
    <div class="tabs" role="tablist" aria-label="Scenario">
      <button class="tab" :class="{ active: mode === 'consistent' }" type="button" @click="mode = 'consistent'">
        Consistent vocabulary
      </button>
      <button class="tab" :class="{ active: mode === 'overloaded' }" type="button" @click="mode = 'overloaded'">
        Overloaded terms
      </button>
    </div>

    <div class="panel">
      <header class="panel-head">
        <h3>{{ scenario.title }}</h3>
        <p>{{ scenario.subtitle }}</p>
      </header>

      <div class="repo">
        <div
          v-for="block in relevantZones"
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

        <div
          v-if="scenario.showAmbiguous"
          v-for="block in ambiguousZones"
          :key="block.id"
          class="rect ambiguous"
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
      <span class="legend-item"><i class="chip relevant" /> Relevant units</span>
      <span class="legend-item"><i class="chip found" /> Found references</span>
      <span class="legend-item"><i class="chip ambiguous" /> Ambiguous terms</span>
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
  background: rgba(22, 163, 74, 0.22);
  border: 2px solid rgba(22, 163, 74, 0.72);
}

.found {
  background: rgba(37, 99, 235, 0.2);
  border: 2px solid rgba(30, 64, 175, 0.68);
  z-index: 2;
}

.ambiguous {
  background: rgba(245, 158, 11, 0.16);
  border: 2px solid rgba(217, 119, 6, 0.62);
  z-index: 3;
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
  background: rgba(22, 163, 74, 0.22);
  border: 2px solid rgba(22, 163, 74, 0.72);
}

.chip.found {
  background: rgba(37, 99, 235, 0.2);
  border: 2px solid rgba(30, 64, 175, 0.68);
}

.chip.ambiguous {
  background: rgba(245, 158, 11, 0.18);
  border: 2px solid rgba(217, 119, 6, 0.62);
}
</style>
