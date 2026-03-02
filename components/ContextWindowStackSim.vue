<script setup lang="ts">
import { computed, ref, watch } from 'vue'

type SegmentKind =
  | 'system'
  | 'tools'
  | 'instructions'
  | 'user'
  | 'assistant'
  | 'toolCall'
  | 'toolOutput'

type Segment = {
  id: string
  label: string
  short: string
  kind: SegmentKind
  size: number
}

const slider = ref(0)
const turn = ref(0)

const turnSnapshots = [0, 18, 36, 54, 76, 100]

const baseSegments: Segment[] = [
  { id: 'system', label: 'System baseline', short: 'System', kind: 'system', size: 4 },
  { id: 'tools', label: 'Tool schemas', short: 'Tools', kind: 'tools', size: 18 },
  { id: 'instr', label: 'Instructions + AGENTS + skills metadata', short: 'Instr', kind: 'instructions', size: 8 },
]

const dynamicSegments: Segment[] = [
  { id: 'u1', label: 'User #1', short: 'U1', kind: 'user', size: 6 },
  { id: 'a1', label: 'Assistant #1', short: 'A1', kind: 'assistant', size: 6 },
  { id: 'tc1', label: 'Tool call #1', short: 'T1c', kind: 'toolCall', size: 2 },
  { id: 'to1', label: 'Tool output #1', short: 'T1o', kind: 'toolOutput', size: 8 },
  { id: 'u2', label: 'User #2', short: 'U2', kind: 'user', size: 5 },
  { id: 'a2', label: 'Assistant #2', short: 'A2', kind: 'assistant', size: 5 },
  { id: 'tc2', label: 'Tool call #2', short: 'T2c', kind: 'toolCall', size: 2 },
  { id: 'to2', label: 'Tool output #2', short: 'T2o', kind: 'toolOutput', size: 7 },
  { id: 'u3', label: 'User #3', short: 'U3', kind: 'user', size: 4 },
  { id: 'a3', label: 'Assistant #3', short: 'A3', kind: 'assistant', size: 4 },
  { id: 'tc3', label: 'Tool call #3', short: 'T3c', kind: 'toolCall', size: 2 },
  { id: 'to3', label: 'Tool output #3', short: 'T3o', kind: 'toolOutput', size: 7 },
  { id: 'u4', label: 'Current user input', short: 'U4', kind: 'user', size: 4 },
  { id: 'a4', label: 'Recent assistant tail', short: 'A4', kind: 'assistant', size: 4 },
  { id: 'tc4', label: 'Tool call #4', short: 'T4c', kind: 'toolCall', size: 2 },
  { id: 'to4', label: 'Tool output #4', short: 'T4o', kind: 'toolOutput', size: 2 },
]

const baseUsed = baseSegments.reduce((sum, s) => sum + s.size, 0)
const dynamicCapacity = 100 - baseUsed

const usedPercent = computed(() => {
  const dynamicUsed = (slider.value / 100) * dynamicCapacity
  return Math.min(100, +(baseUsed + dynamicUsed).toFixed(1))
})

const usedSegments = computed(() => {
  const segments: Segment[] = [...baseSegments]
  let remaining = usedPercent.value - baseUsed

  for (const seg of dynamicSegments) {
    if (remaining <= 0) break
    const piece = Math.min(seg.size, remaining)
    segments.push({ ...seg, size: +piece.toFixed(1) })
    remaining -= piece
  }

  return segments
})

const legend = [
  { kind: 'system', label: 'System baseline' },
  { kind: 'tools', label: 'Tool schemas' },
  { kind: 'instructions', label: 'Instructions / AGENTS / skills metadata' },
  { kind: 'user', label: 'User messages' },
  { kind: 'assistant', label: 'Assistant messages' },
  { kind: 'toolCall', label: 'Tool calls' },
  { kind: 'toolOutput', label: 'Tool outputs' },
]

const middleOpacity = computed(() => {
  const usage = usedPercent.value / 100
  return Math.max(0.08, +(0.56 - usage * 0.42).toFixed(2))
})

const innerMidOpacity = computed(() => {
  const usage = usedPercent.value / 100
  return Math.max(0.03, +(0.42 - usage * 0.36).toFixed(2))
})

const salienceGradient = computed(() => {
  const usage = usedPercent.value / 100
  const edgeTone = 250
  const troughTone = Math.round(164 - usage * 110)
  const stopCount = 25
  const stops = Array.from({ length: stopCount }, (_, index) => {
    const position = index / (stopCount - 1)
    const distanceFromCenter = Math.abs(position - 0.5) / 0.5
    const curve = distanceFromCenter ** 2.35
    const tone = Math.round(troughTone + (edgeTone - troughTone) * curve)
    return `rgb(${tone},${tone},${tone}) ${(position * 100).toFixed(1)}%`
  })

  return `linear-gradient(90deg, ${stops.join(', ')})`
})

const zoneLabel = computed(() => {
  if (usedPercent.value < 40) return 'Smart zone (rule of thumb)'
  if (usedPercent.value < 60) return 'Dumb-zone risk rising (heuristic)'
  return 'High degradation risk'
})

const relevanceBaseByKind: Record<SegmentKind, number> = {
  system: 0.45,
  tools: 0.22,
  instructions: 0.72,
  user: 0.78,
  assistant: 0.6,
  toolCall: 0.3,
  toolOutput: 0.44,
}

const signalSlices = computed(() => {
  const usage01 = usedPercent.value / 100
  const total = usedSegments.value.length

  return usedSegments.value.map((seg, i) => {
    const progress = total <= 1 ? 0 : i / (total - 1)
    const drift = progress * 0.34 * usage01

    const recencyDistance = total - 1 - i
    const recencyBoost =
      recencyDistance <= 2 ? (0.18 - recencyDistance * 0.05) * (0.4 + usage01 * 0.6) : 0

    const deterministicVariance = ((((i + 1) * (turn.value + 3)) % 7) - 3) * 0.015

    const ratio = Math.max(
      0.08,
      Math.min(0.92, relevanceBaseByKind[seg.kind] - drift + recencyBoost + deterministicVariance),
    )
    return {
      id: seg.id,
      width: seg.size,
      relevant: +(seg.size * ratio).toFixed(2),
      noise: +(seg.size * (1 - ratio)).toFixed(2),
      label: seg.label,
    }
  })
})

const noisePercent = computed(() => +signalSlices.value.reduce((sum, s) => sum + s.noise, 0).toFixed(1))

const driftBaseByKind: Record<SegmentKind, number> = {
  system: 0,
  tools: 0,
  instructions: 0,
  user: 0,
  assistant: 0.24,
  toolCall: 0.2,
  toolOutput: 0.3,
}

const failedBaseByKind: Record<SegmentKind, number> = {
  system: 0,
  tools: 0,
  instructions: 0,
  user: 0,
  assistant: 0.18,
  toolCall: 0.08,
  toolOutput: 0.22,
}

const driftEligibleKinds = new Set<SegmentKind>(['assistant', 'toolCall', 'toolOutput'])
const failedEligibleKinds = new Set<SegmentKind>(['assistant', 'toolCall', 'toolOutput'])

function clamp(value: number, min: number, max: number) {
  return Math.min(max, Math.max(min, value))
}

const driftSlices = computed(() => {
  const usage01 = usedPercent.value / 100
  const total = usedSegments.value.length

  return usedSegments.value.map((seg, i) => {
    const signalSlice = signalSlices.value[i]
    const noiseRatio = signalSlice && signalSlice.width > 0 ? signalSlice.noise / signalSlice.width : 0
    const progress = total <= 1 ? 0 : i / (total - 1)
    const age = 1 - progress
    const recencyDistance = total - 1 - i
    const recencyShield =
      recencyDistance <= 2 ? (0.14 - recencyDistance * 0.04) * (0.5 + usage01 * 0.5) : 0

    const deterministicVariance = ((((i + 2) * (turn.value + 5)) % 9) - 4) * 0.01

    let driftRatio = 0
    if (driftEligibleKinds.has(seg.kind)) {
      driftRatio =
        driftBaseByKind[seg.kind] + age * 0.24 * usage01 - recencyShield + deterministicVariance
      driftRatio = clamp(driftRatio, 0, 0.78)
    }
    let failedRatio = 0
    if (failedEligibleKinds.has(seg.kind)) {
      failedRatio =
        failedBaseByKind[seg.kind] +
        age * 0.2 * usage01 -
        recencyShield * 0.55 +
        usage01 * 0.06 +
        deterministicVariance * 0.7
      failedRatio = clamp(failedRatio, 0, 0.72)
    }

    // Drift/failed attempts are modeled as harmful subset of noisy content, never of relevant content.
    const maxCombined = Math.min(0.9, Math.max(0, noiseRatio * 0.95))
    const combined = driftRatio + failedRatio
    if (combined > maxCombined && combined > 0) {
      const scale = maxCombined / combined
      driftRatio *= scale
      failedRatio *= scale
    }

    const stableRatio = Math.max(0, 1 - driftRatio - failedRatio)
    const ratioTotal = stableRatio + driftRatio + failedRatio

    return {
      id: seg.id,
      width: seg.size,
      stable: +(seg.size * (stableRatio / ratioTotal)).toFixed(2),
      drift: +(seg.size * (driftRatio / ratioTotal)).toFixed(2),
      failed: +(seg.size * (failedRatio / ratioTotal)).toFixed(2),
      label: seg.label,
    }
  })
})

const driftPercent = computed(() => +driftSlices.value.reduce((sum, s) => sum + s.drift, 0).toFixed(1))
const failedPercent = computed(() => +driftSlices.value.reduce((sum, s) => sum + s.failed, 0).toFixed(1))

const positionRisk = computed(() => {
  const loss = 1 - innerMidOpacity.value / 0.42
  return +(clamp(loss, 0, 1) * 100).toFixed(1)
})

const noiseRisk = computed(() => {
  const used = Math.max(1, usedPercent.value)
  return +clamp((noisePercent.value / used) * 100, 0, 100).toFixed(1)
})

const driftRisk = computed(() => {
  const used = Math.max(1, usedPercent.value)
  const weighted = driftPercent.value + failedPercent.value * 1.25
  return +clamp((weighted / used) * 100, 0, 100).toFixed(1)
})

const degradationScore = computed(() => {
  const base = positionRisk.value * 0.34 + noiseRisk.value * 0.33 + driftRisk.value * 0.33
  const interaction = (noiseRisk.value / 100) * (driftRisk.value / 100) * 22
  return +clamp(base + interaction, 0, 100).toFixed(1)
})

function sendToLlm() {
  turn.value = (turn.value + 1) % turnSnapshots.length
  slider.value = turnSnapshots[turn.value]
}

const sendButtonLabel = computed(() => (turn.value === turnSnapshots.length - 1 ? 'Reset' : 'Send to LLM'))

watch(slider, (value) => {
  const nearest = turnSnapshots.reduce(
    (best, current, idx) => {
      const distance = Math.abs(current - value)
      if (distance < best.distance) return { index: idx, distance }
      return best
    },
    { index: 0, distance: Number.POSITIVE_INFINITY },
  )
  turn.value = nearest.index
})

function classFor(kind: SegmentKind) {
  return `k-${kind}`
}
</script>

<template>
  <div class="ctx-sim">
    <div class="control-row">
      <div class="slider-panel">
        <label for="fill">Session growth</label>
        <input id="fill" v-model="slider" type="range" min="0" max="100" step="1" />
      </div>

      <div class="status-panel">
        <button class="send-btn" type="button" @click="sendToLlm">{{ sendButtonLabel }}</button>
        <span class="stat">Used: {{ usedPercent.toFixed(1) }}%</span>
        <span class="zone">{{ zoneLabel }}</span>
        <span class="turn">Turn {{ turn + 1 }}/{{ turnSnapshots.length }}</span>
      </div>
    </div>

    <div class="timeline-wrap">
      <div class="timeline-title">Context window</div>
      <div class="legend">
        <span v-for="entry in legend" :key="entry.kind" class="legend-item">
          <span :class="['swatch', classFor(entry.kind)]" />
          {{ entry.label }}
        </span>
      </div>
      <div class="track">
        <template v-for="seg in usedSegments" :key="seg.id">
          <div
            :class="['seg', classFor(seg.kind)]"
            :style="{ width: `${seg.size}%` }"
            :title="`${seg.label} (~${seg.size}%)`"
          />
        </template>
        <div class="free" :style="{ width: `${Math.max(0, 100 - usedPercent)}%` }" title="Free headroom" />
      </div>
    </div>

    <div class="aggregate-block">
      <div class="aggregate-title">Aggregated degradation</div>
      <div class="mini-legend">
        <span class="mini-item"><span class="mini-swatch mini-low" /> Low</span>
        <span class="mini-item"><span class="mini-swatch mini-mid" /> Medium</span>
        <span class="mini-item"><span class="mini-swatch mini-high" /> High</span>
      </div>
      <div class="aggregate-track">
        <div class="aggregate-gradient" />
        <div class="aggregate-mask" :style="{ width: `${Math.max(0, 100 - usedPercent)}%` }" />
      </div>
    </div>

    <div class="signal-block">
      <div class="signal-title">Signal vs noise</div>
      <div class="mini-legend">
        <span class="mini-item"><span class="mini-swatch signal-key-relevant" /> Relevant signal</span>
        <span class="mini-item"><span class="mini-swatch signal-key-noise" /> Noise</span>
      </div>
      <div class="signal-track">
        <template v-for="slice in signalSlices" :key="slice.id">
          <div class="signal-slice" :style="{ width: `${slice.width}%` }" :title="slice.label">
            <div class="signal-relevant" :style="{ width: `${(slice.relevant / slice.width) * 100}%` }" />
            <div class="signal-noise" :style="{ width: `${(slice.noise / slice.width) * 100}%` }" />
          </div>
        </template>
        <div class="signal-free" :style="{ width: `${Math.max(0, 100 - usedPercent)}%` }" />
      </div>
    </div>

    <div class="drift-block">
      <div class="drift-title">Drift + failed attempts</div>
      <div class="mini-legend">
        <span class="mini-item"><span class="mini-swatch mini-stable" /> Stable</span>
        <span class="mini-item"><span class="mini-swatch mini-drift" /> Drift</span>
        <span class="mini-item"><span class="mini-swatch mini-failed" /> Failed attempts</span>
      </div>
      <div class="drift-track">
        <template v-for="slice in driftSlices" :key="slice.id">
          <div class="drift-slice" :style="{ width: `${slice.width}%` }" :title="slice.label">
            <div class="drift-stable" :style="{ width: `${(slice.stable / slice.width) * 100}%` }" />
            <div class="drift-risk" :style="{ width: `${(slice.drift / slice.width) * 100}%` }" />
            <div class="drift-failed" :style="{ width: `${(slice.failed / slice.width) * 100}%` }" />
          </div>
        </template>
        <div class="drift-free" :style="{ width: `${Math.max(0, 100 - usedPercent)}%` }" />
      </div>
    </div>

    <div class="salience-block">
      <div class="salience-title">Lost in the middle</div>
      <div class="mini-legend">
        <span class="mini-item"><span class="mini-swatch salience-high" /> Higher weight</span>
        <span class="mini-item">
          <span class="mini-swatch salience-low" /> Lower weight (less likely remembered)
        </span>
      </div>
      <div class="salience-track">
        <div
          class="salience-used"
          :style="{ width: `${usedPercent}%`, background: salienceGradient }"
          title="Middle gets weaker as context grows"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.ctx-sim {
  --bar-height: 20px;
  --bar-width-transition: none;
  width: 100%;
  display: grid;
  gap: 0.5rem;
}

.control-row {
  display: flex;
  gap: 0.9rem;
  align-items: center;
  font-size: 0.9rem;
}

.slider-panel {
  flex: 1 1 auto;
  min-width: 0;
  display: grid;
  grid-template-columns: max-content minmax(0, 1fr);
  gap: 0.6rem;
  align-items: center;
}

.slider-panel input[type='range'] {
  width: 100%;
}

.status-panel {
  flex: 0 0 auto;
  display: flex;
  gap: 0.6rem;
  align-items: center;
}

.stat {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  font-weight: 700;
  min-width: 8.2rem;
  padding: 0.12rem 0.45rem;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(15, 23, 42, 0.32);
  white-space: nowrap;
}

.zone {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  width: 16.5rem;
  padding: 0.12rem 0.45rem;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.25);
  font-size: 0.8rem;
  opacity: 0.92;
  white-space: nowrap;
}

.send-btn {
  min-width: 7.6rem;
  border: 1px solid rgba(255, 255, 255, 0.32);
  background: #0f172a;
  color: #e5e7eb;
  border-radius: 999px;
  font-size: 0.75rem;
  padding: 0.28rem 0.62rem;
  cursor: pointer;
  text-align: center;
  white-space: nowrap;
}

.send-btn:hover {
  background: #1e293b;
}

.turn {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  font-size: 0.78rem;
  opacity: 0.9;
  min-width: 5.8rem;
  padding: 0.12rem 0.45rem;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(15, 23, 42, 0.22);
  text-align: center;
  white-space: nowrap;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.24rem 0.6rem;
  font-size: 0.68rem;
  opacity: 0.9;
  margin-bottom: 0.16rem;
}

.legend-item {
  display: inline-flex;
  align-items: center;
  gap: 0.26rem;
}

.swatch {
  width: 0.68rem;
  height: 0.68rem;
  border-radius: 3px;
  display: inline-block;
  border: 1px solid rgba(0, 0, 0, 0.2);
}

.timeline-wrap {
  position: relative;
}

.timeline-title {
  font-size: 0.74rem;
  margin-bottom: 0.2rem;
  opacity: 0.9;
}

.track {
  display: flex;
  height: var(--bar-height);
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.24);
  background: #111827;
}

.seg,
.free {
  height: 100%;
  transition: var(--bar-width-transition);
}

.free {
  background: #111827;
}

.salience-block {
  margin-top: 0;
}

.salience-title {
  font-size: 0.74rem;
  margin-bottom: 0.2rem;
  opacity: 0.9;
}

.salience-track {
  height: var(--bar-height);
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.24);
  background: #111827;
}

.salience-used {
  height: 100%;
  transition: var(--bar-width-transition);
}

.signal-block {
  margin-top: 0;
}

.signal-title {
  font-size: 0.74rem;
  margin-bottom: 0.2rem;
  opacity: 0.9;
}

.signal-track {
  height: var(--bar-height);
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.24);
  background: #111827;
  display: flex;
}

.signal-slice {
  height: 100%;
  display: flex;
  transition: var(--bar-width-transition);
}

.signal-relevant {
  height: 100%;
  background: #f59e0b;
  transition: var(--bar-width-transition);
}

.signal-noise {
  height: 100%;
  background: #6b7280;
  transition: var(--bar-width-transition);
}

.signal-free {
  height: 100%;
  background: #111827;
  transition: var(--bar-width-transition);
}

.signal-key-relevant {
  background: #f59e0b;
}

.signal-key-noise {
  background: #6b7280;
}

.drift-block {
  margin-top: 0;
}

.drift-title {
  font-size: 0.74rem;
  margin-bottom: 0.2rem;
  opacity: 0.9;
}

.mini-legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.24rem 0.6rem;
  font-size: 0.68rem;
  opacity: 0.9;
  margin-bottom: 0.16rem;
}

.mini-item {
  display: inline-flex;
  align-items: center;
  gap: 0.26rem;
}

.mini-swatch {
  width: 0.68rem;
  height: 0.68rem;
  border-radius: 3px;
  border: 1px solid rgba(0, 0, 0, 0.2);
  display: inline-block;
}

.mini-stable,
.mini-low {
  background: #34d399;
}

.mini-drift,
.mini-mid {
  background: #f59e0b;
}

.mini-failed,
.mini-high {
  background: #ef4444;
}

.salience-high {
  background: #fafafa;
}

.salience-low {
  background: #9ca3af;
}

.drift-track {
  height: var(--bar-height);
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.24);
  background: #111827;
  display: flex;
}

.drift-slice {
  height: 100%;
  display: flex;
  transition: var(--bar-width-transition);
}

.drift-stable {
  height: 100%;
  background: #34d399;
  transition: var(--bar-width-transition);
}

.drift-risk {
  height: 100%;
  background: #f59e0b;
  transition: var(--bar-width-transition);
}

.drift-failed {
  height: 100%;
  background: #ef4444;
  transition: var(--bar-width-transition);
}

.drift-free {
  height: 100%;
  background: #111827;
  transition: var(--bar-width-transition);
}

.aggregate-block {
  margin-top: 0;
}

.aggregate-title {
  font-size: 0.74rem;
  margin-bottom: 0.2rem;
  opacity: 0.9;
}

.aggregate-track {
  position: relative;
  height: var(--bar-height);
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.24);
  background: #111827;
}

.aggregate-gradient {
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, #34d399 0%, #f59e0b 55%, #ef4444 100%);
}

.aggregate-mask {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  background: #111827;
  transition: var(--bar-width-transition);
}

.k-system {
  background: #bf616a;
}

.k-tools {
  background: #d08770;
}

.k-instructions {
  background: #ebcb8b;
  color: #2e3440;
}

.k-user {
  background: #a3be8c;
  color: #2e3440;
}

.k-assistant {
  background: #8fbcbb;
  color: #2e3440;
}

.k-toolCall {
  background: #b48ead;
}

.k-toolOutput {
  background: #88c0d0;
  color: #2e3440;
}

@media (max-width: 900px) {
  .ctx-sim {
    --bar-height: 22px;
  }

  .control-row {
    flex-direction: column;
    align-items: stretch;
  }

  .slider-panel {
    grid-template-columns: 1fr;
    gap: 0.35rem;
  }

  .status-panel {
    flex-wrap: wrap;
  }

  .zone,
  .stat,
  .turn,
  .send-btn {
    width: auto;
    min-width: 0;
  }

}
</style>
