<script setup lang="ts">
import { computed, ref } from 'vue'

type Mode = 'alwaysOn' | 'taskScoped'

const mode = ref<Mode>('alwaysOn')

const allTools = ['filesystem', 'github', 'jira', 'slack', 'notion', 'browser']
const taskScopedTools = ['filesystem', 'github']

const enabledTools = computed(() => (mode.value === 'alwaysOn' ? allTools : taskScopedTools))
const mutedTools = computed(() => (mode.value === 'taskScoped' ? allTools.filter((tool) => !taskScopedTools.includes(tool)) : []))

const relevantFill = computed(() => (mode.value === 'alwaysOn' ? 36 : 68))
const noiseFill = computed(() => 100 - relevantFill.value)

const signalDensity = computed(() => relevantFill.value)
const qualityProxy = computed(() => (mode.value === 'alwaysOn' ? 44 : 79))
const payloadFill = computed(() => (mode.value === 'alwaysOn' ? 72 : 34))

const status = computed(() => {
  if (mode.value === 'alwaysOn') {
    return 'Always-on mode: extra tool schemas dilute task-relevant signal with noise.'
  }
  return 'Task-scoped mode: irrelevant tool noise is removed, so signal stays dense and outputs are more reliable.'
})
</script>

<template>
  <section class="sim" aria-label="MCP tool tax comparison">
    <div class="toggle-row" role="tablist" aria-label="MCP mode">
      <button class="toggle-btn" :class="{ active: mode === 'alwaysOn' }" type="button" @click="mode = 'alwaysOn'">
        Always-on MCP
      </button>
      <button class="toggle-btn" :class="{ active: mode === 'taskScoped' }" type="button" @click="mode = 'taskScoped'">
        Task-scoped MCP
      </button>
    </div>

    <div class="legend">
      <span><i class="swatch clean" /> task-relevant signal</span>
      <span><i class="swatch noisy" /> irrelevant schema noise</span>
      <span><i class="swatch packet" /> right-sized policy</span>
      <span><i class="swatch muted" /> inactive state</span>
    </div>

    <article class="card active">
      <h3>Loaded context composition</h3>
      <div class="bar-shell">
        <div class="bar clean" :style="{ width: `${relevantFill}%` }" />
        <div class="bar noisy" :style="{ width: `${noiseFill}%` }" />
      </div>
    </article>

    <article class="card active">
      <h3>Signal density for next-token decisions</h3>
      <div class="bar-shell">
        <div class="bar packet" :style="{ width: `${signalDensity}%` }" />
      </div>
    </article>

    <article class="card active">
      <h3>Tool definition payload (tax)</h3>
      <div class="bar-shell">
        <div class="bar noisy" :style="{ width: `${payloadFill}%` }" />
      </div>
    </article>

    <article class="card active">
      <h3>Output quality proxy</h3>
      <div class="bar-shell">
        <div class="bar packet" :style="{ width: `${qualityProxy}%` }" />
      </div>
    </article>

    <article class="card active">
      <h3>Enabled MCP servers</h3>
      <div class="chip-row">
        <span
          v-for="tool in enabledTools"
          :key="`on-${tool}`"
          class="chip"
          :class="mode === 'taskScoped' ? 'packet' : 'noisy'"
        >
          {{ tool }}
        </span>
      </div>
      <div class="chip-row" v-if="mutedTools.length">
        <span v-for="tool in mutedTools" :key="`off-${tool}`" class="chip muted">
          {{ tool }} off
        </span>
      </div>
    </article>

    <p class="status">{{ status }}</p>
  </section>
</template>

<style scoped>
.sim {
  width: min(100%, 14.5rem);
  display: grid;
  gap: 0.22rem;
}

.toggle-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.18rem;
}

.toggle-btn {
  border-radius: 999px;
  border: 2px solid #95a9be;
  background: #e9f0f6;
  color: #1f2937;
  padding: 0.12rem 0.35rem;
  font-size: 0.49rem;
  font-weight: 800;
  cursor: pointer;
}

.toggle-btn.active {
  background: #deecd0;
  border-color: #8bae74;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.3rem;
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

.swatch.noisy { background: #d29266; }
.swatch.clean { background: #84b6d7; }
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

.bar.noisy { background: linear-gradient(90deg, #d29266, #e3b085); }
.bar.clean { background: linear-gradient(90deg, #84b6d7, #b2d6ec); }
.bar.packet { background: linear-gradient(90deg, #8bae74, #b8d6a5); }

.chip-row {
  margin-top: 0.12rem;
  display: flex;
  flex-wrap: wrap;
  gap: 0.12rem;
}

.chip {
  border-radius: 999px;
  padding: 0.08rem 0.22rem;
  font-size: 0.4rem;
  font-weight: 800;
}

.chip.noisy {
  background: #fbe8d9;
  border: 1px solid #d29266;
  color: #7c2d12;
}

.chip.packet {
  background: #edf7e6;
  border: 1px solid #8bae74;
  color: #14532d;
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
