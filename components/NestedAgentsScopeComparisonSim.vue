<script setup lang="ts">
import { computed, ref } from 'vue'

type Scenario = 'notTouching' | 'touching'
type Policy = 'rootHeavy' | 'nestedScoped'
type GuidanceCard = {
  id: string
  label: string
  state: 'relevant' | 'noise' | 'inactive'
}

const scenario = ref<Scenario>('notTouching')
const policy = ref<Policy>('rootHeavy')

const relevantFill = computed(() => {
  if (scenario.value === 'notTouching') return policy.value === 'rootHeavy' ? 38 : 68
  return 64
})

const noiseFill = computed(() => 100 - relevantFill.value)

const densityFill = computed(() => {
  if (scenario.value === 'notTouching') return policy.value === 'rootHeavy' ? 44 : 78
  return 74
})

const guidanceCards = computed<GuidanceCard[]>(() => {
  if (scenario.value === 'notTouching' && policy.value === 'rootHeavy') {
    return [
      { id: 'global', label: 'Global baseline guidance', state: 'relevant' },
      { id: 'repo', label: 'Repo-wide guidance', state: 'relevant' },
      { id: 'domain', label: 'Domain-specific guidance', state: 'noise' },
    ]
  }

  if (scenario.value === 'notTouching' && policy.value === 'nestedScoped') {
    return [
      { id: 'global', label: 'Global baseline guidance', state: 'relevant' },
      { id: 'repo', label: 'Repo-wide guidance', state: 'relevant' },
      { id: 'domain', label: 'Domain-specific guidance', state: 'inactive' },
    ]
  }

  return [
    { id: 'global', label: 'Global baseline guidance', state: 'relevant' },
    { id: 'repo', label: 'Repo-wide guidance', state: 'relevant' },
    { id: 'domain', label: 'Domain-specific guidance', state: 'relevant' },
  ]
})

const status = computed(() => {
  if (scenario.value === 'notTouching' && policy.value === 'rootHeavy') {
    return 'Root-heavy loading pulls in extra domain instructions that are irrelevant for this task.'
  }
  if (scenario.value === 'notTouching' && policy.value === 'nestedScoped') {
    return 'Nested scoping keeps unrelated domain guidance out, improving signal density.'
  }
  return 'When the task touches the domain, both policies load relevant guidance, so practical quality is similar.'
})
</script>

<template>
  <section class="sim" aria-label="Nested agents scope comparison">
    <div class="tab-row" role="tablist" aria-label="Task scenario">
      <button class="tab-btn" :class="{ active: scenario === 'notTouching' }" type="button" @click="scenario = 'notTouching'">
        Not touching domain
      </button>
      <button class="tab-btn" :class="{ active: scenario === 'touching' }" type="button" @click="scenario = 'touching'">
        Touching domain
      </button>
    </div>

    <div class="policy-row" role="tablist" aria-label="Loading policy">
      <button class="policy-btn" :class="{ active: policy === 'rootHeavy' }" type="button" @click="policy = 'rootHeavy'">
        Root-heavy AGENTS
      </button>
      <button class="policy-btn" :class="{ active: policy === 'nestedScoped' }" type="button" @click="policy = 'nestedScoped'">
        Nested-scoped AGENTS
      </button>
    </div>

    <div class="legend">
      <span><i class="swatch clean" /> relevant guidance</span>
      <span><i class="swatch noisy" /> instruction noise</span>
      <span><i class="swatch packet" /> scope clarity</span>
      <span><i class="swatch muted" /> inactive guidance</span>
    </div>

    <article class="card">
      <h3>Loaded instruction composition</h3>
      <div class="bar-shell">
        <div class="bar clean" :style="{ width: `${relevantFill}%` }" />
        <div class="bar noisy" :style="{ width: `${noiseFill}%` }" />
      </div>
    </article>

    <article class="card">
      <h3>Guidance signal density</h3>
      <div class="bar-shell">
        <div class="bar packet" :style="{ width: `${densityFill}%` }" />
      </div>
    </article>

    <article class="card">
      <h3>Loaded guidance chain</h3>
      <div class="chain">
        <span
          v-for="card in guidanceCards"
          :key="card.id"
          class="chip"
          :class="{
            clean: card.state === 'relevant',
            noisy: card.state === 'noise',
            muted: card.state === 'inactive',
          }"
        >
          {{ card.label }}
        </span>
      </div>
    </article>

    <p class="status">{{ status }}</p>
  </section>
</template>

<style scoped>
.sim {
  width: min(100%, 14.8rem);
  display: grid;
  gap: 0.24rem;
}

.tab-row,
.policy-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.18rem;
}

.tab-btn,
.policy-btn {
  border-radius: 999px;
  border: 2px solid #95a9be;
  background: #e9f0f6;
  color: #1f2937;
  padding: 0.12rem 0.35rem;
  font-size: 0.48rem;
  font-weight: 800;
  cursor: pointer;
}

.tab-btn.active,
.policy-btn.active {
  background: #deecd0;
  border-color: #8bae74;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.28rem;
  font-size: 0.44rem;
  font-weight: 700;
  color: #e5e9f0;
  text-shadow: 0 1px 0 rgba(15, 23, 42, 0.35);
}

.swatch {
  width: 0.52rem;
  height: 0.34rem;
  display: inline-block;
  border-radius: 4px;
  margin-right: 0.16rem;
  vertical-align: middle;
}

.swatch.clean { background: #84b6d7; }
.swatch.noisy { background: #d29266; }
.swatch.packet { background: #8bae74; }
.swatch.muted { background: #9ca3af; }

.card {
  border: 1px solid #90a5ba;
  border-radius: 10px;
  background: #f8fbff;
  padding: 0.2rem 0.26rem;
}

.card h3 {
  margin: 0;
  font-size: 0.5rem;
  color: #1f2937;
  font-weight: 800;
}

.bar-shell {
  margin-top: 0.12rem;
  height: 0.4rem;
  border: 1px solid #a8bccf;
  border-radius: 999px;
  background: #e8eef4;
  overflow: hidden;
  display: flex;
}

.bar {
  height: 100%;
  transition: width 220ms ease;
}

.bar.clean { background: linear-gradient(90deg, #84b6d7, #b2d6ec); }
.bar.noisy { background: linear-gradient(90deg, #d29266, #e3b085); }
.bar.packet { background: linear-gradient(90deg, #8bae74, #b8d6a5); }

.chain {
  margin-top: 0.12rem;
  display: flex;
  flex-wrap: wrap;
  gap: 0.12rem;
}

.chip {
  border-radius: 999px;
  padding: 0.08rem 0.2rem;
  font-size: 0.4rem;
  font-weight: 800;
}

.chip.clean {
  background: #e8f2fb;
  border: 1px solid #84b6d7;
  color: #1e3a5f;
}

.chip.noisy {
  background: #fbe8d9;
  border: 1px solid #d29266;
  color: #7c2d12;
}

.chip.muted {
  background: #f2f4f7;
  border: 1px solid #b7bec8;
  color: #4b5563;
}

.status {
  margin: 0.04rem 0 0;
  font-size: 0.44rem;
  line-height: 1.25;
  color: #e6edf5;
  text-shadow: 0 1px 0 rgba(15, 23, 42, 0.35);
  font-weight: 700;
}
</style>
