<script setup lang="ts">
import { computed } from 'vue'

type SegmentKind = 'overhead' | 'signal' | 'noise'

type Segment = {
  kind: SegmentKind
  value: number
}

const props = withDefaults(
  defineProps<{
    usage: number
    segments: Segment[]
    caption?: string
    collapseOverhead?: boolean
  }>(),
  {
    collapseOverhead: false,
  },
)

function clampPercent(value: number) {
  if (!Number.isFinite(value)) return 0
  return Math.max(0, Math.min(100, value))
}

const usage = computed(() => clampPercent(props.usage))

const normalized = computed(() => {
  const targetKinds: SegmentKind[] = props.collapseOverhead ? ['signal', 'noise'] : ['overhead', 'signal', 'noise']

  const sums = new Map<SegmentKind, number>([
    ['overhead', 0],
    ['signal', 0],
    ['noise', 0],
  ])

  for (const seg of props.segments) {
    if (!sums.has(seg.kind)) continue
    const normalizedKind = props.collapseOverhead && seg.kind === 'overhead' ? 'noise' : seg.kind
    const next = (sums.get(normalizedKind) ?? 0) + Math.max(0, seg.value)
    sums.set(normalizedKind, next)
  }

  const total = targetKinds.reduce((acc, k) => acc + (sums.get(k) ?? 0), 0)
  const u = usage.value

  // Scale to usage if caller provided mismatched totals.
  const scale = total > 0 ? u / total : 0

  const items = targetKinds.map((kind) => ({
    kind,
    value: clampPercent((sums.get(kind) ?? 0) * scale),
  }))

  const used = items.reduce((acc, i) => acc + i.value, 0)
  const free = clampPercent(100 - used)

  return { items, used, free }
})

function segStyle(value: number) {
  return { width: `${clampPercent(value)}%` }
}
</script>

<template>
  <section class="cw" aria-label="Context window composition">
    <div class="head">
      <span class="label">Context window</span>
      <span v-if="caption" class="caption">{{ caption }}</span>
    </div>

    <div
      class="track"
      role="img"
      :aria-label="
        collapseOverhead
          ? 'Context window fill: signal, noise, and free headroom'
          : 'Context window fill: overhead, signal, noise, and free headroom'
      "
    >
      <div class="fill" :style="{ width: `${normalized.used}%` }">
        <i
          v-for="seg in normalized.items"
          :key="`seg-${seg.kind}`"
          class="seg"
          :class="seg.kind"
          :style="segStyle(seg.value)"
        />
      </div>
      <div class="free" :style="{ width: `${normalized.free}%` }" />
    </div>

    <div class="legend" aria-label="Legend">
      <span v-if="!collapseOverhead" class="legend-item"><i class="chip overhead" /> Overhead</span>
      <span class="legend-item"><i class="chip signal" /> Signal</span>
      <span class="legend-item"><i class="chip noise" /> Noise</span>
      <span class="legend-item"><i class="chip free" /> Free headroom</span>
    </div>
  </section>
</template>

<style scoped>
.cw {
  display: grid;
  gap: 0.18rem;
}

.head {
  display: flex;
  gap: 0.4rem;
  align-items: baseline;
  justify-content: space-between;
}

.label {
  font-size: 0.56rem;
  font-weight: 900;
  color: #111827;
}

.caption {
  font-size: 0.52rem;
  font-weight: 800;
  color: #475569;
}

.track {
  height: 0.42rem;
  border-radius: 999px;
  background: #e2e8f0;
  overflow: hidden;
  display: flex;
}

.fill {
  height: 100%;
  display: flex;
  transition: width 240ms ease;
  flex: 0 0 auto;
}

.seg {
  height: 100%;
  display: block;
  transition: width 240ms ease;
  flex: 0 0 auto;
}

.seg.overhead {
  background: #93c5fd;
}

.seg.signal {
  background: #86efac;
}

.seg.noise {
  background: #fca5a5;
}

.free {
  height: 100%;
  background: #e2e8f0;
  flex: 0 0 auto;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.45rem;
  align-items: center;
  font-size: 0.52rem;
  color: #000;
  font-weight: 700;
  text-shadow: none;
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

.chip.overhead {
  background: #93c5fd;
}

.chip.signal {
  background: #86efac;
}

.chip.noise {
  background: #fca5a5;
}

.chip.free {
  background: #e2e8f0;
}
</style>
