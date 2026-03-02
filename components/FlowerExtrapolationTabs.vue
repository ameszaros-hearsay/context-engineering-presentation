<script setup lang="ts">
import { computed, ref } from 'vue'

type QualityTab = 'high' | 'low'

const activeTab = ref<QualityTab>('high')
const revealed = ref<Record<QualityTab, boolean>>({
  high: false,
  low: false,
})

const imageSources: Record<QualityTab, string> = {
  high: '/images/flowers-hiq.jpg',
  low: '/images/flowers-lowq.jpg',
}

const isRevealed = computed(() => revealed.value[activeTab.value])
const activeImage = computed(() => imageSources[activeTab.value])
const actionLabel = computed(() => (isRevealed.value ? 'reset' : 'extrapolate'))

function setTab(tab: QualityTab) {
  activeTab.value = tab
}

function toggleExtrapolation() {
  const tab = activeTab.value
  revealed.value[tab] = !revealed.value[tab]
}
</script>

<template>
  <section class="flower-sim" aria-label="Image extrapolation quality comparison">
    <div class="image-shell">
      <img :src="activeImage" :alt="activeTab === 'high' ? 'high quality flowers' : 'low quality flowers'" class="flower-img" />
      <div v-if="!isRevealed" class="right-cover" aria-hidden="true" />
    </div>

    <div class="controls">
      <div class="tabs" role="tablist" aria-label="Image quality tabs">
        <button
          class="tab"
          :class="{ active: activeTab === 'high' }"
          type="button"
          @click="setTab('high')"
        >
          high quality
        </button>
        <button
          class="tab"
          :class="{ active: activeTab === 'low' }"
          type="button"
          @click="setTab('low')"
        >
          low quality
        </button>
      </div>

      <button class="action-btn" type="button" @click="toggleExtrapolation">
        {{ actionLabel }}
      </button>
    </div>
  </section>
</template>

<style scoped>
.flower-sim {
  width: min(100%, 860px);
  margin: 0 auto;
  display: grid;
  gap: 0.34rem;
}

.image-shell {
  position: relative;
  width: 100%;
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid #9fb4c8;
  background: #dde7f1;
  aspect-ratio: 16 / 6;
}

.flower-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.right-cover {
  position: absolute;
  top: 0;
  right: 0;
  width: 33.3334%;
  height: 100%;
  background:
    linear-gradient(180deg, rgba(232, 240, 248, 0.98), rgba(217, 230, 243, 0.98)),
    repeating-linear-gradient(
      135deg,
      rgba(118, 139, 162, 0.16) 0px,
      rgba(118, 139, 162, 0.16) 8px,
      rgba(118, 139, 162, 0.04) 8px,
      rgba(118, 139, 162, 0.04) 16px
    );
  border-left: 1px dashed rgba(65, 88, 111, 0.6);
}

.controls {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.45rem;
  flex-wrap: wrap;
}

.tabs {
  display: flex;
  gap: 0.36rem;
}

.tab,
.action-btn {
  border-radius: 999px;
  border: 2px solid #90a8bf;
  background: #e9f1f8;
  color: #1f2937;
  padding: 0.12rem 0.44rem;
  font-size: 0.54rem;
  font-weight: 800;
  cursor: pointer;
}

.tab.active {
  border-color: #6d94bc;
  background: #d6e6f6;
}

.action-btn {
  border-color: #7e9f67;
  background: #deecd0;
}

@media (max-width: 900px) {
  .image-shell {
    aspect-ratio: 16 / 7;
  }

  .controls {
    justify-content: center;
  }
}
</style>
