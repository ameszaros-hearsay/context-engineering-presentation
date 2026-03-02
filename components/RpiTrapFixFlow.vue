<script setup lang="ts">
import { computed, ref } from 'vue'

const props = withDefaults(
  defineProps<{
    highlight?: 'trap' | 'fix' | 'both'
  }>(),
  {},
)

const localMode = ref<'trap' | 'fix' | 'both'>('both')
const mode = computed(() => props.highlight ?? localMode.value)

function setMode(next: 'trap' | 'fix' | 'both') {
  if (props.highlight) return
  localMode.value = next
}

function laneClass(lane: 'trap' | 'fix') {
  if (mode.value === 'both') return 'on'
  return mode.value === lane ? 'on' : 'off'
}
</script>

<template>
  <section class="flow" aria-label="Trap versus Fix execution flow">
    <div class="controls" role="tablist">
      <button class="chip" :class="{ active: mode === 'trap' }" type="button" @click="setMode('trap')">
        Trap
      </button>
      <button class="chip" :class="{ active: mode === 'fix' }" type="button" @click="setMode('fix')">
        Fix
      </button>
      <button class="chip" :class="{ active: mode === 'both' }" type="button" @click="setMode('both')">
        Both
      </button>
    </div>

    <div class="lane trap" :class="laneClass('trap')">
      <strong>The Trap</strong>
      <div class="steps">
        <span>Append “fix this”</span>
        <span>Compound bad history</span>
        <span>Hallucination risk</span>
      </div>
    </div>

    <div class="lane fix" :class="laneClass('fix')">
      <strong>The Fix</strong>
      <div class="steps">
        <span>Revert quickly</span>
        <span>Repair the plan</span>
        <span>Rerun with clean history</span>
      </div>
    </div>
  </section>
</template>

<style scoped>
.flow {
  display: grid;
  gap: 0.28rem;
}

.controls {
  display: flex;
  gap: 0.3rem;
  justify-content: flex-end;
}

.chip {
  border-radius: 999px;
  border: 2px solid #9bb0c4;
  background: #e8f0f8;
  color: #0f172a;
  padding: 0.1rem 0.4rem;
  font-size: 0.52rem;
  font-weight: 800;
  cursor: pointer;
}

.chip.active {
  border-color: #2563eb;
  background: #dbeafe;
}

.lane {
  border: 2px solid;
  border-radius: 12px;
  padding: 0.24rem 0.32rem;
  transition: opacity 120ms ease;
}

.lane.off {
  opacity: 0.35;
}

.lane.on {
  opacity: 1;
}

.lane strong {
  display: block;
  font-size: 0.7rem;
  font-weight: 900;
  margin-bottom: 0.16rem;
}

.trap {
  border-color: #fca5a5;
  background: #fff1f2;
  color: #7f1d1d;
}

.fix {
  border-color: #86efac;
  background: #f0fdf4;
  color: #14532d;
}

.steps {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 0.2rem;
}

.steps span {
  border-radius: 8px;
  border: 2px solid currentColor;
  background: rgba(255, 255, 255, 0.65);
  padding: 0.2rem 0.25rem;
  font-size: 0.54rem;
  font-weight: 800;
  line-height: 1.2;
  text-align: center;
}
</style>
