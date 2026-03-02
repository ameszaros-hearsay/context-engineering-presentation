<script setup lang="ts">
import { computed, ref } from 'vue'

const mode = ref<'artifact' | 'raw'>('artifact')

const mainLoad = computed(() => (mode.value === 'artifact' ? 18 : 58))
const driftRisk = computed(() => (mode.value === 'artifact' ? 'Low' : 'High'))
</script>

<template>
  <section class="lanes" aria-label="Subagent context isolation flow">
    <div class="controls">
      <button class="btn" :class="{ active: mode === 'artifact' }" type="button" @click="mode = 'artifact'">
        Distilled artifact
      </button>
      <button class="btn" :class="{ active: mode === 'raw' }" type="button" @click="mode = 'raw'">
        Raw logs (anti-pattern)
      </button>
      <span class="metric">Main load: {{ mainLoad }} units</span>
      <span class="metric">Drift risk: {{ driftRisk }}</span>
    </div>

    <div class="lane-grid">
      <article class="lane main" :class="{ polluted: mode === 'raw' }">
        <h3>Main thread</h3>
        <ul>
          <li>Keep scoped objective and constraints</li>
          <li>Delegate broad exploration out-of-band</li>
          <li>Integrate only what affects implementation</li>
          <li>Stay clean for coding decisions</li>
        </ul>
      </article>

      <article class="lane research">
        <h3>Subagent research lane</h3>
        <ul>
          <li>Run broad file/tool discovery</li>
          <li>Check alternatives and dead ends</li>
          <li>Compress findings into one artifact</li>
          <li>Send high-signal handoff back</li>
        </ul>
      </article>
    </div>

    <div class="bridge" :class="{ polluted: mode === 'raw' }">
      <span v-if="mode === 'artifact'">Parent thread receives: distilled artifact only</span>
      <span v-else>Parent thread receives: raw traces, dead ends, and noise</span>
    </div>
  </section>
</template>

<style scoped>
.lanes {
  display: grid;
  gap: 0.36rem;
  margin-top: 0.24rem;
  color: #1f2937;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.3rem;
}

.btn {
  border: 2px solid #97abc0;
  border-radius: 999px;
  background: #e8eef4;
  color: #1f2937;
  padding: 0.16rem 0.48rem;
  font-size: 0.57rem;
  font-weight: 800;
  cursor: pointer;
}

.btn.active {
  background: #deecd0;
  border-color: #8bae74;
}

.metric {
  border: 1px solid #a1b5c8;
  border-radius: 999px;
  padding: 0.13rem 0.4rem;
  background: #f8fbff;
  font-size: 0.54rem;
  font-weight: 700;
  color: #334155;
}

.lane-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.34rem;
}

.lane {
  border: 2px solid #9eb2c6;
  border-radius: 12px;
  background: #edf3f8;
  padding: 0.33rem 0.42rem;
}

.lane.main {
  background: #d9ebf8;
  border-color: #84b6d7;
}

.lane.main.polluted {
  background: #fbe8d9;
  border-color: #d29266;
}

.lane.research {
  background: #deecd0;
  border-color: #8bae74;
}

.lane h3 {
  margin: 0;
  font-size: 0.66rem;
  font-weight: 800;
  color: #1f2937;
}

.lane ul {
  margin: 0.2rem 0 0;
  padding-left: 0.78rem;
  display: grid;
  gap: 0.11rem;
}

.lane li {
  font-size: 0.56rem;
  line-height: 1.15;
  color: #334155;
}

.bridge {
  border: 2px dashed #86b08a;
  border-radius: 10px;
  background: #eef7e9;
  padding: 0.2rem 0.4rem;
  text-align: center;
  font-size: 0.56rem;
  font-weight: 800;
  color: #14532d;
}

.bridge.polluted {
  border-color: #d29266;
  background: #fff1e7;
  color: #7c2d12;
}

@media (max-width: 980px) {
  .lane-grid {
    grid-template-columns: 1fr;
  }
}
</style>
