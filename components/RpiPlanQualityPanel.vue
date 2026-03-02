<script setup lang="ts">
import { computed, ref } from 'vue'

const props = withDefaults(
  defineProps<{
    initialMode?: 'good' | 'bad'
  }>(),
  {
    initialMode: 'good',
  },
)

const mode = ref<'good' | 'bad'>(props.initialMode)

const model = computed(() => {
  if (mode.value === 'bad') {
    return {
      title: 'Bad plan posture',
      summary: 'Ambiguous steps produce wide diffs and expensive rework.',
      implementationQuality: 38,
      reworkRisk: 82,
      reviewability: 32,
      tone: 'bad',
      quote: 'A bad plan is easier to fix than bad code at scale.',
    }
  }

  return {
    title: 'Good plan posture',
    summary: 'Explicit files, steps, and tests keep execution constrained.',
    implementationQuality: 84,
    reworkRisk: 28,
    reviewability: 86,
    tone: 'good',
    quote: 'A clear plan prevents expensive implementation drift.',
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
  <section class="panel" aria-label="Plan quality simulator">
    <div class="controls" role="tablist">
      <button class="chip" :class="{ active: mode === 'good' }" type="button" @click="mode = 'good'">
        Good plan
      </button>
      <button class="chip" :class="{ active: mode === 'bad' }" type="button" @click="mode = 'bad'">
        Bad plan
      </button>
    </div>

    <article class="card" :class="model.tone">
      <h4>{{ model.title }}</h4>
      <p>{{ model.summary }}</p>

      <div class="metric">
        <span>Implementation quality</span>
        <div class="track"><i :style="barStyle(model.implementationQuality, '#60a5fa')" /></div>
      </div>

      <div class="metric">
        <span>Rework risk</span>
        <div class="track"><i :style="barStyle(model.reworkRisk, '#f87171')" /></div>
      </div>

      <div class="metric">
        <span>Reviewability</span>
        <div class="track"><i :style="barStyle(model.reviewability, '#4ade80')" /></div>
      </div>
    </article>

    <aside class="statement" :class="model.tone">
      {{ model.quote }}
    </aside>
  </section>
</template>

<style scoped>
.panel {
  width: 100%;
  max-width: 12rem;
  display: grid;
  gap: 0.28rem;
}

.controls {
  display: flex;
  gap: 0.3rem;
  justify-content: center;
}

.chip {
  border-radius: 999px;
  border: 2px solid #9bb0c4;
  background: #e8f0f8;
  color: #0f172a;
  padding: 0.1rem 0.4rem;
  font-size: 0.54rem;
  font-weight: 800;
  cursor: pointer;
}

.chip.active {
  border-color: #2563eb;
  background: #dbeafe;
}

.card {
  border-radius: 12px;
  border: 2px solid #90a6bb;
  background: #f8fbff;
  padding: 0.32rem 0.36rem;
}

.card.good {
  border-color: #16a34a;
}

.card.bad {
  border-color: #ef4444;
}

.card h4 {
  margin: 0;
  font-size: 0.68rem;
  font-weight: 900;
  color: #0f172a;
}

.card p {
  margin: 0.08rem 0 0.2rem;
  font-size: 0.54rem;
  color: #475569;
}

.metric {
  display: grid;
  grid-template-columns: 5.7rem 1fr;
  align-items: center;
  gap: 0.2rem;
  margin-top: 0.11rem;
}

.metric span {
  font-size: 0.52rem;
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

.statement {
  border-radius: 12px;
  border: 2px solid;
  padding: 0.34rem 0.4rem;
  font-size: 0.6rem;
  font-weight: 900;
  line-height: 1.28;
  text-align: left;
}

.statement.good {
  border-color: #86efac;
  background: #f0fdf4;
  color: #14532d;
}

.statement.bad {
  border-color: #fca5a5;
  background: #fef2f2;
  color: #7f1d1d;
}
</style>
