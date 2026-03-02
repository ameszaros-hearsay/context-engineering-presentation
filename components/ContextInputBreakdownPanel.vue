<script setup lang="ts">
const developerItems = ['4.1 shell sandbox/permissions', '4.2 extra dev instructions']
const projectItems = [
  '4.3 global AGENTS',
  '4.4 repo AGENTS chain',
  '4.5 skills metadata',
  '4.6 env context',
]
const memoryItems = ['4.7 chat history + tool traces', 'assistant answers + tool outputs']
</script>

<template>
  <div class="input-breakdown">
    <section class="input-shell">
      <div class="shell-head">
        <span class="code-tag">[INPUT[]]</span>
        <span class="title-text">conversation items (oldest -&gt; newest)</span>
      </div>

      <div class="group">
        <div class="group-head developer">Developer layer</div>
        <div class="items" v-click="1">
          <div v-for="item in developerItems" :key="item" class="item">{{ item }}</div>
        </div>
      </div>

      <div class="group">
        <div class="group-head project">Project/user layer</div>
        <div class="items two-cols" v-click="2">
          <div v-for="item in projectItems" :key="item" class="item">{{ item }}</div>
        </div>
      </div>

      <div class="group">
        <div class="group-head memory">Conversation memory</div>
        <div class="items two-cols" v-click="3">
          <div v-for="item in memoryItems" :key="item" class="item">{{ item }}</div>
        </div>
      </div>
    </section>
  </div>
</template>

<style scoped>
.input-breakdown {
  --text-main: #1f2937;
  --text-muted: #334155;
  --shell-bg: #e7edf4;
  --shell-border: #8ea6bb;
  --dev-bg: #d9ebf8;
  --dev-border: #86b8d9;
  --project-bg: #deecd0;
  --project-border: #96bc7e;
  --memory-bg: #f6ddca;
  --memory-border: #d8a37f;
  color: var(--text-main);
}

.input-shell {
  border-radius: 16px;
  border: 2px solid var(--shell-border);
  background: var(--shell-bg);
  padding: 0.65rem 0.75rem 0.65rem;
  display: grid;
  gap: 0.45rem;
}

.shell-head {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 1rem;
  font-weight: 800;
  color: var(--text-main);
}

.title-text {
  color: var(--text-main);
}

.group {
  display: grid;
  gap: 0.28rem;
}

.group-head {
  border-radius: 12px;
  border: 2px solid #9eaebf;
  padding: 0.22rem 0.5rem;
  font-size: 0.82rem;
  font-weight: 800;
  color: var(--text-main);
}

.group-head.developer {
  background: var(--dev-bg);
  border-color: var(--dev-border);
}

.group-head.project {
  background: var(--project-bg);
  border-color: var(--project-border);
}

.group-head.memory {
  background: var(--memory-bg);
  border-color: var(--memory-border);
}

.items {
  display: grid;
  gap: 0.24rem;
}

.items.two-cols {
  grid-template-columns: 1fr 1fr;
}

.item {
  border-radius: 10px;
  border: 1px solid #9db0c3;
  background: #f8fafc;
  padding: 0.16rem 0.45rem;
  font-size: 0.65rem;
  font-weight: 700;
  color: var(--text-main);
}

.code-tag {
  border-radius: 8px;
  background: #1f2937;
  color: #d7f3ff;
  font-size: 0.74em;
  padding: 0.08em 0.45em;
  border: 1px solid #5b7288;
  font-weight: 800;
}
</style>
