<script setup lang="ts">
import { computed, ref } from 'vue'

type SegmentKind = 'system' | 'tools' | 'instructions' | 'history' | 'summary'

type Segment = {
  id: string
  label: string
  short: string
  kind: SegmentKind
  size: number
}

type SignalSlice = {
  id: string
  label: string
  width: number
  signal: number
  noise: number
}

type Scenario = {
  key: 'before' | 'after'
  title: string
  subtitle: string
  used: number
  segments: Segment[]
  signalSlices: SignalSlice[]
  badge?: string
  reliability: 'Low reliability' | 'High reliability'
  reliabilityTone: 'low' | 'high'
  reliabilityNote: string
}

const tab = ref<'before' | 'after'>('before')

const scenarios: Record<'before' | 'after', Scenario> = {
  before: {
    key: 'before',
    title: 'Session is almost full',
    subtitle: 'Most of the window is occupied by accumulated turns and retries.',
    used: 92,
    segments: [
      { id: 'system', label: 'System baseline', short: 'System', kind: 'system', size: 9 },
      { id: 'tools', label: 'Tool definitions', short: 'Tools', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', short: 'Instr', kind: 'instructions', size: 10 },
      { id: 'history-1', label: 'Older session history', short: 'Hist', kind: 'history', size: 25 },
      { id: 'history-2', label: 'Retries and dead branches', short: 'Retries', kind: 'history', size: 17 },
      { id: 'history-3', label: 'Recent turns', short: 'Recent', kind: 'history', size: 15 },
    ],
    signalSlices: [
      { id: 'system', label: 'System baseline', width: 9, signal: 5, noise: 4 },
      { id: 'tools', label: 'Tool definitions', width: 16, signal: 4, noise: 12 },
      { id: 'instructions', label: 'Instructions', width: 10, signal: 7, noise: 3 },
      { id: 'history-1', label: 'Older session history', width: 25, signal: 9, noise: 16 },
      { id: 'history-2', label: 'Retries and dead branches', width: 17, signal: 2, noise: 15 },
      { id: 'history-3', label: 'Recent turns', width: 15, signal: 8, noise: 7 },
    ],
    reliability: 'Low reliability',
    reliabilityTone: 'low',
    reliabilityNote: 'Continuation is conditioned by a crowded, noisy history.',
  },
  after: {
    key: 'after',
    title: 'Compacted session restores headroom',
    subtitle: 'Fixed context remains, noisy history drops, and the retained handoff becomes denser.',
    used: 49,
    segments: [
      { id: 'system', label: 'System baseline', short: 'System', kind: 'system', size: 9 },
      { id: 'tools', label: 'Tool definitions', short: 'Tools', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', short: 'Instr', kind: 'instructions', size: 10 },
      { id: 'summary', label: 'Compacted handoff', short: 'Handoff', kind: 'summary', size: 14 },
    ],
    signalSlices: [
      { id: 'system', label: 'System baseline', width: 9, signal: 5, noise: 4 },
      { id: 'tools', label: 'Tool definitions', width: 16, signal: 4, noise: 12 },
      { id: 'instructions', label: 'Instructions', width: 10, signal: 7, noise: 3 },
      { id: 'summary', label: 'Compacted handoff', width: 14, signal: 13, noise: 1 },
    ],
    reliability: 'High reliability',
    reliabilityTone: 'high',
    reliabilityNote: 'The model sees cleaner retained context with restored headroom.',
  },
}

const active = computed(() => scenarios[tab.value])

function classFor(kind: SegmentKind) {
  return `k-${kind}`
}
</script>

<template>
  <section class="compaction-sim" aria-label="Compaction before and after">
    <div class="tabs" role="tablist" aria-label="Compaction state">
      <button
        class="tab"
        :class="{ active: tab === 'before' }"
        type="button"
        @click="tab = 'before'"
      >
        Before compaction
      </button>
      <button
        class="tab"
        :class="{ active: tab === 'after' }"
        type="button"
        @click="tab = 'after'"
      >
        Compacted session
      </button>
    </div>

    <div class="panel">
      <header class="panel-head">
        <h3>{{ active.title }}</h3>
        <p>{{ active.subtitle }}</p>
      </header>

      <div class="reliability" :class="`reliability-${active.reliabilityTone}`">
        <div class="reliability-label">If the user continues here</div>
        <div class="reliability-value">{{ active.reliability }}</div>
        <div class="reliability-note">{{ active.reliabilityNote }}</div>
      </div>

      <div class="block">
        <div class="row-head">
          <span>Context window</span>
          <span class="usage">Used {{ active.used }}%</span>
        </div>
        <div class="legend bar-legend" aria-label="Context window legend">
          <span class="legend-item"><i class="chip k-system" /> System</span>
          <span class="legend-item"><i class="chip k-tools" /> Tool definitions</span>
          <span class="legend-item"><i class="chip k-instructions" /> Instructions</span>
          <span class="legend-item"><i class="chip k-history" /> Full session history</span>
          <span class="legend-item"><i class="chip k-summary" /> Compacted handoff</span>
        </div>
        <div class="track" aria-label="Context window usage">
          <template v-for="segment in active.segments" :key="segment.id">
            <div
              :class="['segment', classFor(segment.kind)]"
              :style="{ width: `${segment.size}%` }"
              :title="segment.label"
            />
          </template>
          <div class="free" :style="{ width: `${100 - active.used}%` }" title="Free headroom" />
        </div>
      </div>

      <div class="block">
        <div class="row-head">
          <span>Signal vs noise</span>
          <span class="hint">Orange = useful retained context</span>
        </div>
        <div class="legend bar-legend" aria-label="Signal vs noise legend">
          <span class="legend-item"><i class="chip signal-key" /> Signal</span>
          <span class="legend-item"><i class="chip noise-key" /> Noise</span>
        </div>
        <div class="signal-track" aria-label="Signal vs noise">
          <template v-for="slice in active.signalSlices" :key="slice.id">
            <div class="signal-slice" :style="{ width: `${slice.width}%` }" :title="slice.label">
              <div class="signal" :style="{ width: `${(slice.signal / slice.width) * 100}%` }" />
              <div class="noise" :style="{ width: `${(slice.noise / slice.width) * 100}%` }" />
            </div>
          </template>
          <div class="signal-free" :style="{ width: `${100 - active.used}%` }" />
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
.compaction-sim {
  display: grid;
  gap: 0.35rem;
  margin-top: 0.5rem;
}

.tabs {
  display: flex;
  gap: 0.36rem;
  flex-wrap: wrap;
}

.tab {
  border-radius: 999px;
  border: 2px solid #8ea3b8;
  background: #e9f0f6;
  color: #1f2937;
  padding: 0.16rem 0.52rem;
  font-size: 0.62rem;
  font-weight: 800;
  cursor: pointer;
}

.tab.active {
  border-color: #c7770a;
  background: #fff2d8;
}

.panel {
  border: 1px solid #8ea3b8;
  border-radius: 12px;
  background: #f8fbff;
  padding: 0.42rem 0.46rem 0.4rem;
  display: grid;
  gap: 0.38rem;
}

.panel-head h3 {
  margin: 0;
  font-size: 0.78rem;
  font-weight: 800;
  color: #1f2937;
}

.panel-head p {
  margin: 0.06rem 0 0;
  font-size: 0.6rem;
  color: #475569;
}

.reliability {
  border-radius: 10px;
  border: 1px solid #b6c7d6;
  padding: 0.28rem 0.36rem;
  display: grid;
  gap: 0.02rem;
}

.reliability-low {
  background: #fff1f2;
  border-color: #f0b2bb;
}

.reliability-high {
  background: #ecfdf5;
  border-color: #9bd3b1;
}

.reliability-label {
  font-size: 0.5rem;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  color: #64748b;
}

.reliability-value {
  font-size: 0.76rem;
  font-weight: 900;
  color: #111827;
}

.reliability-note {
  font-size: 0.54rem;
  color: #475569;
}

.block {
  display: grid;
  gap: 0.14rem;
}

.row-head {
  display: flex;
  justify-content: space-between;
  gap: 0.5rem;
  align-items: center;
  font-size: 0.58rem;
  font-weight: 700;
  color: #334155;
}

.usage,
.hint {
  color: #64748b;
  font-weight: 700;
}

.track,
.signal-track {
  height: 1.2rem;
  border-radius: 10px;
  overflow: hidden;
  border: 1px solid #b6c7d6;
  background: #111827;
  display: flex;
}

.segment,
.free,
.signal-free {
  height: 100%;
}

.segment {
  display: block;
}

.free,
.signal-free {
  background: #111827;
}

.signal-slice {
  position: relative;
  display: flex;
  height: 100%;
}

.signal,
.noise {
  height: 100%;
}

.signal {
  background: #f59e0b;
}

.noise {
  background: #6b7280;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.28rem 0.55rem;
  align-items: center;
  font-size: 0.5rem;
  color: #475569;
  font-weight: 700;
}

.bar-legend {
  margin-bottom: 0.04rem;
}

.legend-item {
  display: inline-flex;
  align-items: center;
  gap: 0.24rem;
}

.chip {
  width: 0.58rem;
  height: 0.58rem;
  border-radius: 4px;
  display: inline-block;
  border: 1px solid rgba(15, 23, 42, 0.25);
}

.signal-key {
  background: #f59e0b;
}

.noise-key {
  background: #6b7280;
}

.k-system {
  background: #bf616a;
}

.k-tools {
  background: #d08770;
}

.k-instructions {
  background: #ebcb8b;
}

.k-history {
  background: #8fbcbb;
}

.k-summary {
  background: #f6ad55;
}
</style>
