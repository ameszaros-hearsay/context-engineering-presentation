<script setup lang="ts">
type Layer = {
  index: string
  tag: string
  label: string
  tone: 'system' | 'tools' | 'instr' | 'input'
}

const layers: Layer[] = [
  { index: '1', tag: '[SYSTEM]', label: 'OpenAI baseline layer', tone: 'system' },
  { index: '2', tag: '[TOOLS]', label: 'Built-in and MCP schemas', tone: 'tools' },
  { index: '3', tag: '[INSTR]', label: 'Config instruction layer', tone: 'instr' },
  { index: '4', tag: '[INPUT[]]', label: 'Conversation items stream', tone: 'input' },
]
</script>

<template>
  <div class="aha">
    <div class="source-row">
      <section class="source-card system">
        <p class="kicker">OpenAI platform</p>
        <p class="main-line">
          <span class="code-tag">[SYSTEM]</span>
          baseline
        </p>
      </section>

      <section class="source-card runtime">
        <p class="kicker">Codex runtime (CLI/app)</p>
        <p class="main-line plain">Builds request each turn</p>
        <div class="runtime-tags">
          <span class="pill tools">[TOOLS]</span>
          <span class="plus">+</span>
          <span class="pill instr">[INSTR]</span>
          <span class="plus">+</span>
          <span class="pill input">[INPUT[]]</span>
        </div>
      </section>
    </div>

    <div class="join-row">
      <div class="join">This becomes one ordered prompt</div>
      <div class="priority">priority: system &gt; developer &gt; user &gt; assistant</div>
    </div>

    <div class="stack" aria-label="Final context order">
      <div v-for="layer in layers" :key="layer.index" class="bar" :class="layer.tone">
        <span class="index">{{ layer.index }}</span>
        <span class="code-tag">{{ layer.tag }}</span>
        <span class="label">{{ layer.label }}</span>
      </div>
    </div>

  </div>
</template>

<style scoped>
.aha {
  --bg-card: #f8fafc;
  --text-main: #1f2937;
  --text-muted: #334155;
  --system-bg: #f6ddca;
  --system-border: #d8a37f;
  --tools-bg: #deecd0;
  --tools-border: #94b97d;
  --instr-bg: #d9ebf8;
  --instr-border: #85b7d9;
  --input-bg: #e5ecf3;
  --input-border: #9fb3c6;
  display: grid;
  gap: 0.6rem;
  color: var(--text-main);
}

.source-row {
  display: grid;
  grid-template-columns: 1fr 1.35fr;
  gap: 0.6rem;
}

.source-card {
  border-radius: 14px;
  border: 2px solid #9aa8bb;
  background: var(--bg-card);
  padding: 0.65rem 0.8rem;
}

.source-card.system {
  border-color: var(--system-border);
  background: var(--system-bg);
}

.source-card.runtime {
  border-color: #a4b7c9;
}

.kicker {
  margin: 0;
  font-size: 0.78rem;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.03em;
  font-weight: 700;
}

.main-line {
  margin: 0.2rem 0 0 0;
  font-size: 1.1rem;
  font-weight: 800;
  color: var(--text-main);
  display: flex;
  gap: 0.55rem;
  align-items: center;
}

.main-line.plain {
  gap: 0;
}

.runtime-tags {
  margin-top: 0.42rem;
  display: flex;
  align-items: center;
  gap: 0.4rem;
  font-weight: 700;
}

.pill {
  border-radius: 999px;
  border: 2px solid #8ea1b3;
  padding: 0.2rem 0.6rem;
  font-size: 0.8rem;
  color: #1f2937;
}

.pill.tools {
  background: var(--tools-bg);
  border-color: var(--tools-border);
}

.pill.instr {
  background: var(--instr-bg);
  border-color: var(--instr-border);
}

.pill.input {
  background: var(--input-bg);
  border-color: var(--input-border);
}

.plus {
  font-size: 1rem;
  font-weight: 800;
  color: var(--text-main);
}

.join-row {
  display: grid;
  grid-template-columns: 1fr auto;
  align-items: center;
  gap: 0.45rem;
}

.join {
  border: 2px dashed #8ea1b3;
  border-radius: 12px;
  padding: 0.24rem 0.55rem;
  text-align: center;
  font-size: 0.92rem;
  font-weight: 700;
  color: #e5e9f0;
  background: #3b4b61;
}

.stack {
  display: grid;
  gap: 0.3rem;
}

.bar {
  border-radius: 12px;
  border: 2px solid #93a8bc;
  background: #f8fafc;
  padding: 0.22rem 0.52rem;
  display: flex;
  align-items: center;
  gap: 0.65rem;
  font-size: 1.02rem;
  font-weight: 700;
  color: var(--text-main);
}

.bar.system {
  background: var(--system-bg);
  border-color: var(--system-border);
}

.bar.tools {
  background: var(--tools-bg);
  border-color: var(--tools-border);
}

.bar.instr {
  background: var(--instr-bg);
  border-color: var(--instr-border);
}

.bar.input {
  background: var(--input-bg);
  border-color: var(--input-border);
}

.index {
  color: #4b5563;
  font-weight: 800;
  min-width: 1ch;
}

.label {
  color: var(--text-main);
}

.code-tag {
  border-radius: 8px;
  background: #1f2937;
  color: #d7f3ff;
  font-size: 0.9em;
  padding: 0.08em 0.45em;
  border: 1px solid #5b7288;
  font-weight: 800;
}

.priority {
  border-radius: 8px;
  border: 1px solid #8da3b7;
  background: #34475d;
  color: #e5e9f0;
  padding: 0.16rem 0.45rem;
  font-size: 0.68rem;
  font-weight: 700;
  white-space: nowrap;
}
</style>
