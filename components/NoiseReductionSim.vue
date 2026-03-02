<script setup lang="ts">
import { computed, ref } from 'vue'
import ContextWindowCompositionBar from './ContextWindowCompositionBar.vue'

type RectKind = 'active' | 'dead' | 'found' | 'edit'

type Rect = {
  id: string
  kind: RectKind
  x: number
  y: number
  w: number
  h: number
}

type Boundary = {
  x: number
  y: number
  w: number
  h: number
}

const mode = ref<'maintained' | 'noisy'>('maintained')

const maintainedBoundary: Boundary = { x: 14, y: 12, w: 72, h: 76 }
const noisyBoundary: Boundary = { x: 4, y: 6, w: 92, h: 88 }

const maintainedActive: Rect[] = [
  { id: 'a-1', kind: 'active', x: 20, y: 18, w: 26, h: 18 },
  { id: 'a-2', kind: 'active', x: 54, y: 22, w: 26, h: 18 },
  { id: 'a-3', kind: 'active', x: 28, y: 58, w: 26, h: 18 },
]

const maintainedDead: Rect[] = [
  { id: 'd-1', kind: 'dead', x: 18, y: 42, w: 16, h: 14 },
  { id: 'd-2', kind: 'dead', x: 66, y: 54, w: 14, h: 12 },
]

const maintainedFound: Rect[] = [
  { id: 'f-1', kind: 'found', x: 18, y: 16, w: 32, h: 22 },
  { id: 'f-2', kind: 'found', x: 50, y: 20, w: 34, h: 22 },
  { id: 'f-3', kind: 'found', x: 26, y: 56, w: 34, h: 22 },
]

const maintainedEdits: Rect[] = [
  { id: 'e-1', kind: 'edit', x: 26, y: 20, w: 14, h: 12 },
  { id: 'e-2', kind: 'edit', x: 58, y: 24, w: 14, h: 12 },
  { id: 'e-3', kind: 'edit', x: 34, y: 60, w: 14, h: 12 },
]

const noisyActive: Rect[] = [
  // Relevant code stays disjoint from dead code blocks (no set-logic contradictions).
  { id: 'a-1', kind: 'active', x: 70, y: 14, w: 20, h: 14 },
  { id: 'a-2', kind: 'active', x: 70, y: 34, w: 20, h: 14 },
  { id: 'a-3', kind: 'active', x: 70, y: 66, w: 20, h: 14 },
]

const noisyDead: Rect[] = [
  { id: 'd-1', kind: 'dead', x: 4, y: 8, w: 60, h: 22 },
  { id: 'd-2', kind: 'dead', x: 4, y: 34, w: 60, h: 22 },
  { id: 'd-3', kind: 'dead', x: 4, y: 60, w: 60, h: 22 },
]

const noisyFound: Rect[] = [
  { id: 'f-1', kind: 'found', x: 4, y: 4, w: 58, h: 34 },
  { id: 'f-2', kind: 'found', x: 34, y: 34, w: 62, h: 34 },
  { id: 'f-3', kind: 'found', x: 8, y: 58, w: 52, h: 34 },
]

const noisyEdits: Rect[] = [
  // Edits spill into dead code zones.
  { id: 'e-1', kind: 'edit', x: 18, y: 14, w: 16, h: 10 },
  { id: 'e-2', kind: 'edit', x: 20, y: 66, w: 18, h: 10 },
  { id: 'e-3', kind: 'edit', x: 72, y: 38, w: 14, h: 10 },
]

const scenario = computed(() => {
  if (mode.value === 'noisy') {
    return {
      title: 'Noisy repo',
      subtitle: 'Dead code is large, searchable, and editable by accident.',
      boundary: noisyBoundary,
      active: noisyActive,
      dead: noisyDead,
      found: noisyFound,
      edits: noisyEdits,
      usage: 86,
      segments: [
        { kind: 'overhead' as const, value: 18 },
        { kind: 'signal' as const, value: 16 },
        { kind: 'noise' as const, value: 52 },
      ],
      caption: 'Big fill, noise dominates',
    }
  }

  return {
    title: 'Maintained repo',
    subtitle: 'Dead code exists, but it stays small and irrelevant.',
    boundary: maintainedBoundary,
    active: maintainedActive,
    dead: maintainedDead,
    found: maintainedFound,
    edits: maintainedEdits,
    usage: 52,
    segments: [
      { kind: 'overhead' as const, value: 18 },
      { kind: 'signal' as const, value: 28 },
      { kind: 'noise' as const, value: 6 },
    ],
    caption: 'More headroom, less pollution',
  }
})

function toAbs(boundary: Boundary, rect: Rect) {
  return {
    left: `${boundary.x + (rect.x * boundary.w) / 100}%`,
    top: `${boundary.y + (rect.y * boundary.h) / 100}%`,
    width: `${(rect.w * boundary.w) / 100}%`,
    height: `${(rect.h * boundary.h) / 100}%`,
  }
}
</script>

<template>
  <section class="sim" aria-label="Noise reduction good vs bad">
    <div class="tabs" role="tablist" aria-label="Scenario">
      <button class="tab" :class="{ active: mode === 'maintained' }" type="button" @click="mode = 'maintained'">
        Maintained repo
      </button>
      <button class="tab" :class="{ active: mode === 'noisy' }" type="button" @click="mode = 'noisy'">
        Noisy repo
      </button>
    </div>

    <div class="panel">
      <header class="panel-head">
        <h3>{{ scenario.title }}</h3>
        <p>{{ scenario.subtitle }}</p>
      </header>

      <div class="repo">
        <div class="boundary" :style="{ left: `${scenario.boundary.x}%`, top: `${scenario.boundary.y}%`, width: `${scenario.boundary.w}%`, height: `${scenario.boundary.h}%` }" />

        <div
          v-for="block in scenario.active"
          :key="block.id"
          class="rect active"
          :style="toAbs(scenario.boundary, block)"
        />

        <div
          v-for="block in scenario.dead"
          :key="block.id"
          class="rect dead"
          :style="toAbs(scenario.boundary, block)"
        />

        <div
          v-for="block in scenario.found"
          :key="block.id"
          class="rect found"
          :style="toAbs(scenario.boundary, block)"
        />

        <div
          v-for="block in scenario.edits"
          :key="block.id"
          class="rect edit"
          :style="toAbs(scenario.boundary, block)"
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
      <span class="legend-item"><i class="chip active" /> Relevant code</span>
      <span class="legend-item"><i class="chip dead" /> Dead code</span>
      <span class="legend-item"><i class="chip found" /> Found references</span>
      <span class="legend-item"><i class="chip edit" /> Agent edits</span>
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

.boundary {
  position: absolute;
  border-radius: 10px;
  border: 2px dashed rgba(15, 23, 42, 0.35);
  background: rgba(15, 23, 42, 0.03);
  pointer-events: none;
}

.rect {
  position: absolute;
  border-radius: 6px;
}

.active {
  background: rgba(22, 163, 74, 0.2);
  border: 2px solid rgba(22, 163, 74, 0.66);
}

.dead {
  background: rgba(100, 116, 139, 0.18);
  border: 2px solid rgba(71, 85, 105, 0.55);
}

.found {
  background: rgba(37, 99, 235, 0.18);
  border: 2px solid rgba(30, 64, 175, 0.62);
}

.edit {
  background: rgba(249, 115, 22, 0.08);
  border: 2px solid rgba(249, 115, 22, 0.9);
  box-shadow: 0 0 0 1px rgba(15, 23, 42, 0.12) inset;
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

.chip.active {
  background: rgba(22, 163, 74, 0.2);
  border: 2px solid rgba(22, 163, 74, 0.66);
}

.chip.dead {
  background: rgba(100, 116, 139, 0.18);
  border: 2px solid rgba(71, 85, 105, 0.55);
}

.chip.found {
  background: rgba(37, 99, 235, 0.18);
  border: 2px solid rgba(30, 64, 175, 0.62);
}

.chip.edit {
  background: rgba(249, 115, 22, 0.08);
  border: 2px solid rgba(249, 115, 22, 0.9);
}
</style>
