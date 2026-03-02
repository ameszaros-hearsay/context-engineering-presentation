<script setup lang="ts">
import { computed, ref } from 'vue'

const mode = ref<'reset' | 'continue'>('reset')

const scenario = computed(() => {
  if (mode.value === 'continue') {
    return {
      usage: 78,
      noise: 68,
      drift: 62,
      reliability: 34,
      label: 'Continue past 40%',
      summary: 'History grows, noise compounds, drift probability rises.',
      tone: 'bad',
    }
  }

  return {
    usage: 32,
    noise: 24,
    drift: 18,
    reliability: 82,
    label: 'Reset at 40%',
    summary: 'Compact + clean session restores focus and reliability.',
    tone: 'good',
  }
})

function barStyle(value: number, color: string) {
  return {
    width: `${value}%`,
    background: color,
  }
}
</script>

<template>
  <section class="sim" aria-label="40 percent reset tradeoff">
    <div class="tabs" role="tablist">
      <button class="tab" :class="{ active: mode === 'reset' }" type="button" @click="mode = 'reset'">
        Reset at 40%
      </button>
      <button class="tab" :class="{ active: mode === 'continue' }" type="button" @click="mode = 'continue'">
        Continue past 40%
      </button>
    </div>

    <article class="card" :class="scenario.tone">
      <h4>{{ scenario.label }}</h4>
      <p>{{ scenario.summary }}</p>

      <div class="metric">
        <span>Context usage</span>
        <div class="track"><i :style="barStyle(scenario.usage, '#93c5fd')" /></div>
      </div>

      <div class="metric">
        <span>Noise load</span>
        <div class="track"><i :style="barStyle(scenario.noise, '#fca5a5')" /></div>
      </div>

      <div class="metric">
        <span>Drift risk</span>
        <div class="track"><i :style="barStyle(scenario.drift, '#fdba74')" /></div>
      </div>

      <div class="metric">
        <span>Expected reliability</span>
        <div class="track"><i :style="barStyle(scenario.reliability, '#86efac')" /></div>
      </div>
    </article>
  </section>
</template>

<style scoped>
.sim {
  display: grid;
  gap: 0.34rem;
}

.tabs {
  display: flex;
  gap: 0.32rem;
  flex-wrap: wrap;
}

.tab {
  border-radius: 999px;
  border: 2px solid #94a3b8;
  background: #e2e8f0;
  color: #1e293b;
  padding: 0.1rem 0.45rem;
  font-size: 0.56rem;
  font-weight: 800;
  cursor: pointer;
}

.tab.active {
  border-color: #22c55e;
  background: #dcfce7;
}

.card {
  border-radius: 12px;
  border: 2px solid #8ea3b8;
  background: #f8fbff;
  padding: 0.28rem 0.36rem;
}

.card.good {
  border-color: #22c55e;
}

.card.bad {
  border-color: #ef4444;
}

.card h4 {
  margin: 0;
  font-size: 0.68rem;
  font-weight: 900;
  color: #1f2937;
}

.card p {
  margin: 0.08rem 0 0.2rem;
  font-size: 0.54rem;
  color: #475569;
}

.metric {
  display: grid;
  grid-template-columns: 6.7rem 1fr;
  align-items: center;
  gap: 0.24rem;
  margin-top: 0.1rem;
}

.metric span {
  font-size: 0.54rem;
  font-weight: 700;
  color: #334155;
}

.track {
  height: 0.34rem;
  border-radius: 999px;
  background: #e2e8f0;
  overflow: hidden;
}

.track i {
  display: block;
  height: 100%;
  border-radius: 999px;
}
</style>
