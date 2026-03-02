<script setup lang="ts">
import { computed, ref } from 'vue'

type NodeId = 'user' | 'cw' | 'ctrl' | 'llm' | 'tool'
type EdgeId =
  | 'user_ctrl'
  | 'cw_ctrl'
  | 'ctrl_llm'
  | 'llm_ctrl'
  | 'ctrl_cw'
  | 'ctrl_tool'
  | 'tool_ctrl'
  | 'ctrl_user'

type SegmentKind =
  | 'system'
  | 'tools'
  | 'instructions'
  | 'user'
  | 'assistant'
  | 'toolCall'
  | 'toolOutput'

type Segment = {
  id: string
  label: string
  kind: SegmentKind
  size: number
}

type Step = {
  node: NodeId
  edge?: EdgeId
  label: string
  context: Segment[]
}

const steps: Step[] = [
  {
    node: 'user',
    label: 'User sends request to controller',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
    ],
  },
  {
    node: 'ctrl',
    edge: 'user_ctrl',
    label: 'Controller receives user message',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
    ],
  },
  {
    node: 'cw',
    edge: 'ctrl_cw',
    label: 'Controller appends user message to context',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
    ],
  },
  {
    node: 'ctrl',
    edge: 'cw_ctrl',
    label: 'Controller reads full context',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
    ],
  },
  {
    node: 'llm',
    edge: 'ctrl_llm',
    label: 'Controller calls LLM with context',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
    ],
  },
  {
    node: 'ctrl',
    edge: 'llm_ctrl',
    label: 'LLM returns output to controller',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
    ],
  },
  {
    node: 'cw',
    edge: 'ctrl_cw',
    label: 'Controller appends assistant plan to context',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant plan', kind: 'assistant', size: 10 },
    ],
  },
  {
    node: 'ctrl',
    edge: 'cw_ctrl',
    label: 'Controller reads full context',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant plan', kind: 'assistant', size: 10 },
    ],
  },
  {
    node: 'tool',
    edge: 'ctrl_tool',
    label: 'Tool call',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant message', kind: 'assistant', size: 10 },
    ],
  },
  {
    node: 'ctrl',
    edge: 'tool_ctrl',
    label: 'Tool result to controller',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant plan', kind: 'assistant', size: 10 },
    ],
  },
  {
    node: 'cw',
    edge: 'ctrl_cw',
    label: 'Controller appends tool result to context',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant plan', kind: 'assistant', size: 10 },
      { id: 'tool-call', label: 'Tool call', kind: 'toolCall', size: 4 },
      { id: 'tool-output', label: 'Tool output', kind: 'toolOutput', size: 12 },
    ],
  },
  {
    node: 'ctrl',
    edge: 'cw_ctrl',
    label: 'Controller reads full context',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant plan', kind: 'assistant', size: 10 },
      { id: 'tool-call', label: 'Tool call', kind: 'toolCall', size: 4 },
      { id: 'tool-output', label: 'Tool output', kind: 'toolOutput', size: 12 },
    ],
  },
  {
    node: 'llm',
    edge: 'ctrl_llm',
    label: 'Controller calls LLM again with updated context',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant plan', kind: 'assistant', size: 10 },
      { id: 'tool-call', label: 'Tool call', kind: 'toolCall', size: 4 },
      { id: 'tool-output', label: 'Tool output', kind: 'toolOutput', size: 12 },
    ],
  },
  {
    node: 'ctrl',
    edge: 'llm_ctrl',
    label: 'LLM returns refined output to controller',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant plan', kind: 'assistant', size: 10 },
      { id: 'tool-call', label: 'Tool call', kind: 'toolCall', size: 4 },
      { id: 'tool-output', label: 'Tool output', kind: 'toolOutput', size: 12 },
    ],
  },
  {
    node: 'cw',
    edge: 'ctrl_cw',
    label: 'Controller appends refined answer to context',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant plan', kind: 'assistant', size: 10 },
      { id: 'tool-call', label: 'Tool call', kind: 'toolCall', size: 4 },
      { id: 'tool-output', label: 'Tool output', kind: 'toolOutput', size: 12 },
      { id: 'assistant-refined', label: 'Refined assistant answer', kind: 'assistant', size: 11 },
    ],
  },
  {
    node: 'ctrl',
    edge: 'cw_ctrl',
    label: 'Controller reads full context',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant plan', kind: 'assistant', size: 10 },
      { id: 'tool-call', label: 'Tool call', kind: 'toolCall', size: 4 },
      { id: 'tool-output', label: 'Tool output', kind: 'toolOutput', size: 12 },
      { id: 'assistant-refined', label: 'Refined assistant answer', kind: 'assistant', size: 11 },
    ],
  },
  {
    node: 'user',
    edge: 'ctrl_user',
    label: 'Controller returns final output to user',
    context: [
      { id: 'system', label: 'System baseline', kind: 'system', size: 10 },
      { id: 'tools', label: 'Tool definitions', kind: 'tools', size: 16 },
      { id: 'instructions', label: 'Instructions', kind: 'instructions', size: 12 },
      { id: 'user', label: 'User message', kind: 'user', size: 8 },
      { id: 'assistant', label: 'Assistant plan', kind: 'assistant', size: 10 },
      { id: 'tool-call', label: 'Tool call', kind: 'toolCall', size: 4 },
      { id: 'tool-output', label: 'Tool output', kind: 'toolOutput', size: 12 },
      { id: 'assistant-refined', label: 'Refined assistant answer', kind: 'assistant', size: 11 },
    ],
  },
]

const index = ref(0)

const current = computed(() => steps[index.value])
const isAtFinal = computed(() => index.value === steps.length - 1)
const usedPercent = computed(() => current.value.context.reduce((sum, seg) => sum + seg.size, 0))
const actionLabel = computed(() => (isAtFinal.value ? 'Reset' : 'Step'))

const legend = [
  { kind: 'system' as const, label: 'System' },
  { kind: 'tools' as const, label: 'Tools' },
  { kind: 'instructions' as const, label: 'Instructions' },
  { kind: 'user' as const, label: 'User' },
  { kind: 'assistant' as const, label: 'Assistant' },
  { kind: 'toolCall' as const, label: 'Tool call' },
  { kind: 'toolOutput' as const, label: 'Tool output' },
]

function stepForward() {
  if (isAtFinal.value) {
    reset()
    return
  }
  index.value = Math.min(index.value + 1, steps.length - 1)
}

function reset() {
  index.value = 0
}

function isNodeActive(node: NodeId) {
  return current.value.node === node
}

function isEdgeActive(edge: EdgeId) {
  return current.value.edge === edge
}

function classFor(kind: SegmentKind) {
  return `k-${kind}`
}
</script>

<template>
  <div class="agent-loop">
    <svg viewBox="0 0 920 430" class="diagram" role="img" aria-label="Agent execution loop simulation">
      <defs>
        <marker id="arrowMuted" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="8" markerHeight="8" orient="auto-start-reverse">
          <path d="M 0 0 L 10 5 L 0 10 z" class="arrow-muted" />
        </marker>
        <marker id="arrowActive" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="8" markerHeight="8" orient="auto-start-reverse">
          <path d="M 0 0 L 10 5 L 0 10 z" class="arrow-active" />
        </marker>
      </defs>

      <!-- Edges -->
      <line x1="260" y1="205" x2="360" y2="205" :class="['edge', { active: isEdgeActive('cw_ctrl') }]" />
      <line x1="360" y1="235" x2="260" y2="235" :class="['edge', { active: isEdgeActive('ctrl_cw') }]" />
      <line x1="450" y1="170" x2="450" y2="95" :class="['edge', { active: isEdgeActive('ctrl_llm') }]" />
      <line x1="490" y1="95" x2="490" y2="170" :class="['edge', { active: isEdgeActive('llm_ctrl') }]" />
      <line x1="560" y1="205" x2="660" y2="205" :class="['edge', { active: isEdgeActive('ctrl_tool') }]" />
      <line x1="660" y1="235" x2="560" y2="235" :class="['edge', { active: isEdgeActive('tool_ctrl') }]" />
      <line x1="430" y1="250" x2="430" y2="335" :class="['edge', { active: isEdgeActive('ctrl_user') }]" />
      <line x1="470" y1="335" x2="470" y2="250" :class="['edge', { active: isEdgeActive('user_ctrl') }]" />

      <!-- Nodes -->
      <rect x="60" y="170" width="200" height="70" rx="14" :class="['node', 'context', { active: isNodeActive('cw') }]" />
      <text x="160" y="205" text-anchor="middle" class="node-title">Context</text>

      <rect x="360" y="170" width="200" height="80" rx="10" :class="['node', { active: isNodeActive('ctrl') }]" />
      <text x="460" y="218" text-anchor="middle" class="node-title">Loop Controller</text>

      <rect x="370" y="20" width="200" height="70" rx="10" :class="['node', { active: isNodeActive('llm') }]" />
      <text x="470" y="55" text-anchor="middle" class="node-title">LLM</text>

      <rect x="660" y="170" width="190" height="80" rx="10" :class="['node', { active: isNodeActive('tool') }]" />
      <text x="755" y="218" text-anchor="middle" class="node-title">Tool Call</text>

      <rect x="350" y="335" width="200" height="78" rx="10" :class="['node', { active: isNodeActive('user') }]" />
      <text x="450" y="374" text-anchor="middle" class="node-title">User</text>
    </svg>

    <div class="context-panel">
      <div class="context-head">
        <span class="context-title">Context window</span>
        <span class="context-usage">Used {{ usedPercent }}%</span>
      </div>

      <div class="legend" aria-label="Context window legend">
        <span v-for="entry in legend" :key="entry.kind" class="legend-item">
          <i :class="['chip', classFor(entry.kind)]" />
          {{ entry.label }}
        </span>
      </div>

      <div class="track" aria-label="Current context window composition">
        <div
          v-for="segment in current.context"
          :key="segment.id"
          :class="['seg', classFor(segment.kind)]"
          :style="{ width: `${segment.size}%` }"
          :title="segment.label"
        />
        <div class="free" :style="{ width: `${Math.max(0, 100 - usedPercent)}%` }" title="Free headroom" />
      </div>
    </div>

    <div class="controls">
      <button class="btn" @click="stepForward">{{ actionLabel }}</button>
      <span class="status">{{ current.label }}</span>
    </div>
  </div>
</template>

<style scoped>
.agent-loop {
  width: 100%;
}

.diagram {
  width: 100%;
  height: auto;
}

.context-panel {
  margin-top: 0.4rem;
  display: grid;
  gap: 0.2rem;
}

.context-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 0.6rem;
}

.context-title {
  font-size: 0.62rem;
  font-weight: 900;
  color: #e2e8f0;
}

.context-usage {
  font-size: 0.58rem;
  font-weight: 800;
  color: #cbd5e1;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.24rem 0.5rem;
  font-size: 0.5rem;
  color: #cbd5e1;
  font-weight: 700;
}

.legend-item {
  display: inline-flex;
  align-items: center;
  gap: 0.24rem;
}

.chip {
  width: 0.52rem;
  height: 0.52rem;
  border-radius: 4px;
  display: inline-block;
  box-shadow: 0 0 0 1px rgba(15, 23, 42, 0.4);
}

.track {
  height: 0.95rem;
  border-radius: 999px;
  overflow: hidden;
  border: 1px solid rgba(147, 162, 184, 0.7);
  background: #0f172a;
  display: flex;
}

.seg,
.free {
  height: 100%;
}

.seg {
  display: block;
}

.free {
  background: #0f172a;
}

.edge {
  stroke: #8aa0ba;
  stroke-width: 3;
  fill: none;
  marker-end: url(#arrowMuted);
}

.edge.active {
  stroke: #60a5fa;
  stroke-width: 5;
  marker-end: url(#arrowActive);
}

.arrow-muted {
  fill: #8aa0ba;
}

.arrow-active {
  fill: #60a5fa;
}

.node {
  fill: #f8fafc;
  stroke: #93a2b8;
  stroke-width: 2;
}

.node.context {
  fill: #e7eef5;
}

.node.active {
  stroke: #60a5fa;
  stroke-width: 4;
  fill: #dbeafe;
  filter: drop-shadow(0 0 6px rgba(96, 165, 250, 0.4));
}

.node-title {
  font-size: 17px;
  fill: #0f172a;
  font-weight: 600;
  dominant-baseline: middle;
}

.node-sub {
  font-size: 13px;
  fill: #334155;
}

.controls {
  margin-top: 8px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.btn {
  background: #60a5fa;
  color: #0f172a;
  border: none;
  padding: 4px 10px;
  border-radius: 7px;
  font-weight: 600;
  font-size: 16px;
  line-height: 1.2;
  cursor: pointer;
}

.status {
  font-size: 16px;
  line-height: 1.2;
  color: #cbd5e1;
}

.k-system {
  background: #bf616a;
}

.k-tools {
  background: #d08770;
}

.k-instructions {
  background: #ebcb8b;
}

.k-user {
  background: #a3be8c;
}

.k-assistant {
  background: #8fbcbb;
}

.k-toolCall {
  background: #81a1c1;
}

.k-toolOutput {
  background: #5e81ac;
}
</style>
