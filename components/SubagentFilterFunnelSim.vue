<script setup lang="ts">
import { computed, ref, watch } from 'vue'

type Tab = 'baseline' | 'delegate'
type BaselinePhase = 1 | 2
type DelegatePhase = 1 | 2 | 3

const tab = ref<Tab>('baseline')
const baselinePhase = ref<BaselinePhase>(1)
const delegatePhase = ref<DelegatePhase>(1)

watch(tab, (next) => {
  if (next === 'baseline') baselinePhase.value = 1
  if (next === 'delegate') delegatePhase.value = 1
})

const rawTokens = ['file A', 'file B', 'dead end', 'tool log', 'alt path']
const packetTokens = ['scope', 'facts', 'constraints']

const baselineMainFill = computed(() => (baselinePhase.value === 1 ? 26 : 86))
const delegateSubFill = computed(() => (delegatePhase.value >= 2 ? 78 : 0))
const delegateMainFill = computed(() => {
  if (delegatePhase.value === 1) return 26
  if (delegatePhase.value === 2) return 26
  return 38
})

const status = computed(() => {
  if (tab.value === 'baseline') {
    if (baselinePhase.value === 1) return 'Baseline / Phase 1: main has existing task context and decides to research.'
    return 'Baseline / Phase 2: main performs broad research itself, so its context window grows heavily.'
  }

  if (delegatePhase.value === 1) return 'Delegate / Phase 1: same starting point as baseline (existing task context in main).'
  if (delegatePhase.value === 2) return 'Delegate / Phase 2: subagent performs broad research in its own context window.'
  return 'Delegate / Phase 3: subagent filters findings into one compact packet and hands it to main.'
})
</script>

<template>
  <section class="sim" aria-label="Subagent filtering walkthrough">
    <div class="tab-row" role="tablist" aria-label="Scenario">
      <button class="tab-btn" :class="{ active: tab === 'baseline' }" type="button" @click="tab = 'baseline'">Baseline</button>
      <button class="tab-btn" :class="{ active: tab === 'delegate' }" type="button" @click="tab = 'delegate'">Delegate</button>
    </div>

    <div class="phase-row" v-if="tab === 'baseline'" role="tablist" aria-label="Baseline phases">
      <button class="phase-btn" :class="{ active: baselinePhase === 1 }" type="button" @click="baselinePhase = 1">Phase 1: Decide</button>
      <button class="phase-btn" :class="{ active: baselinePhase === 2 }" type="button" @click="baselinePhase = 2">Phase 2: Main Research</button>
    </div>

    <div class="phase-row" v-else role="tablist" aria-label="Delegate phases">
      <button class="phase-btn" :class="{ active: delegatePhase === 1 }" type="button" @click="delegatePhase = 1">Phase 1: Decide</button>
      <button class="phase-btn" :class="{ active: delegatePhase === 2 }" type="button" @click="delegatePhase = 2">Phase 2: Subagent Research</button>
      <button class="phase-btn" :class="{ active: delegatePhase === 3 }" type="button" @click="delegatePhase = 3">Phase 3: Handoff</button>
    </div>

    <div class="legend">
      <span><i class="swatch noisy" /> raw research tokens</span>
      <span><i class="swatch packet" /> compact packet</span>
      <span><i class="swatch clean" /> existing task context</span>
    </div>

    <div class="bars" v-if="tab === 'baseline'">
      <article class="bar-card active">
        <h3>Main context window (main does research)</h3>
        <div class="bar-shell">
          <div class="bar" :class="baselinePhase === 2 ? 'noisy' : 'clean'" :style="{ width: `${baselineMainFill}%` }" />
        </div>
        <div class="token-row" v-if="baselinePhase === 1">
          <span class="token clean">existing task context</span>
        </div>
        <div class="token-row" v-else>
          <span class="token clean">existing task context</span>
          <span class="token noisy" v-for="token in rawTokens" :key="`bm-${token}`">{{ token }}</span>
        </div>
      </article>
    </div>

    <div class="bars" v-else>
      <article class="bar-card active">
        <h3>Main context window (delegate path)</h3>
        <div class="bar-shell">
          <div class="bar clean" :style="{ width: `${delegateMainFill}%` }" />
        </div>
        <div class="token-row" v-if="delegatePhase <= 2">
          <span class="token clean">existing task context</span>
        </div>
        <div class="token-row" v-else>
          <span class="token clean">existing task context</span>
          <span class="token packet" v-for="token in packetTokens" :key="`dp-${token}`">{{ token }}</span>
        </div>
      </article>

      <article class="bar-card" :class="{ active: delegatePhase === 2 }">
        <h3>Subagent context window (research lane)</h3>
        <div class="bar-shell">
          <div class="bar noisy" :style="{ width: `${delegateSubFill}%` }" />
        </div>
        <div class="token-row" v-if="delegatePhase >= 2">
          <span class="token noisy" v-for="token in rawTokens" :key="`ds-${token}`">{{ token }}</span>
        </div>
      </article>
    </div>

    <div class="flow" v-if="tab === 'delegate' && delegatePhase >= 2">
      <span class="node sub">Subagent</span>
      <span class="arrow">filters</span>
      <span class="node packet">Packet</span>
      <span class="arrow">sends to</span>
      <span class="node main">Main</span>
    </div>

    <p class="status">{{ status }}</p>
  </section>
</template>

<style scoped>
.sim {
  display: grid;
  gap: 0.28rem;
  margin-top: 0.06rem;
}

.tab-row,
.phase-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.2rem;
}

.tab-btn,
.phase-btn {
  border-radius: 999px;
  border: 2px solid #95a9be;
  background: #e9f0f6;
  color: #1f2937;
  padding: 0.12rem 0.35rem;
  font-size: 0.49rem;
  font-weight: 800;
  cursor: pointer;
}

.tab-btn.active,
.phase-btn.active {
  background: #deecd0;
  border-color: #8bae74;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.3rem;
  font-size: 0.46rem;
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

.swatch.noisy { background: #d29266; }
.swatch.packet { background: #8bae74; }
.swatch.clean { background: #84b6d7; }

.bars {
  display: grid;
  gap: 0.2rem;
}

.bar-card {
  border: 1px solid #90a5ba;
  border-radius: 10px;
  background: #f8fbff;
  padding: 0.2rem 0.26rem;
  opacity: 0.45;
  transition: opacity 180ms ease, box-shadow 180ms ease;
}

.bar-card.active {
  opacity: 1;
  box-shadow: inset 0 0 0 2px rgba(132, 182, 215, 0.45);
}

.bar-card h3 {
  margin: 0;
  font-size: 0.52rem;
  color: #1f2937;
  font-weight: 800;
}

.bar-shell {
  margin-top: 0.14rem;
  height: 0.42rem;
  border: 1px solid #a8bccf;
  border-radius: 999px;
  background: #e8eef4;
  overflow: hidden;
}

.bar {
  height: 100%;
  transition: width 240ms ease;
}

.bar.noisy { background: linear-gradient(90deg, #d29266, #e3b085); }
.bar.clean { background: linear-gradient(90deg, #84b6d7, #b2d6ec); }

.token-row {
  margin-top: 0.12rem;
  display: flex;
  flex-wrap: wrap;
  gap: 0.12rem;
}

.token {
  border-radius: 999px;
  padding: 0.08rem 0.22rem;
  font-size: 0.42rem;
  font-weight: 800;
}

.token.noisy {
  background: #fbe8d9;
  border: 1px solid #d29266;
  color: #7c2d12;
}

.token.packet {
  background: #edf7e6;
  border: 1px solid #8bae74;
  color: #14532d;
}

.token.clean {
  background: #e8f2fb;
  border: 1px solid #84b6d7;
  color: #1e3a5f;
}

.flow {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.14rem;
  justify-content: center;
}

.node {
  border-radius: 999px;
  padding: 0.09rem 0.25rem;
  font-size: 0.44rem;
  font-weight: 800;
}

.node.sub,
.node.packet {
  background: #edf7e6;
  border: 1px solid #8bae74;
  color: #14532d;
}

.node.main {
  background: #e8f2fb;
  border: 1px solid #84b6d7;
  color: #1e3a5f;
}

.arrow {
  font-size: 0.44rem;
  font-weight: 800;
  color: #dce7f5;
  text-shadow: 0 1px 0 rgba(15, 23, 42, 0.35);
}

.status {
  margin: 0;
  text-align: center;
  font-size: 0.5rem;
  font-weight: 800;
  color: #e5e9f0;
  text-shadow: 0 1px 0 rgba(15, 23, 42, 0.35);
}
</style>
