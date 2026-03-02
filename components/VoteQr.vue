<script setup lang="ts">
import { computed } from 'vue'
import { renderSVG } from 'uqr'

type Props = {
  url: string
  /**
   * QR size in px (rendered on slide).
   * @default 180
   */
  size?: number
}

const props = withDefaults(defineProps<Props>(), {
  size: 180,
})

const ariaLabel = 'Vote QR code'

const svg = computed(() => {
  const raw = renderSVG(props.url, {
    ecc: 'M',
    border: 2,
    pixelSize: 4,
    blackColor: 'var(--vote-qr-fg)',
    whiteColor: 'var(--vote-qr-bg)',
  })

  return raw.replace(
    /^<svg\b([^>]*)>/,
    `<svg$1 class="vote-qr__img" role="img" aria-label="${ariaLabel}" shape-rendering="crispEdges">`,
  )
})
</script>

<template>
  <div class="vote-qr" :style="{ '--vote-qr-size': `${size}px` }">
    <div class="vote-qr__svg" v-html="svg" />
  </div>
</template>

<style scoped>
.vote-qr {
  --vote-qr-bg: var(--slidev-bg);
  --vote-qr-fg: #fff;
  --vote-qr-size: 180px;
  display: inline-block;
}

.vote-qr__svg {
  overflow: hidden;
}

.vote-qr__svg :deep(.vote-qr__img) {
  width: var(--vote-qr-size);
  height: var(--vote-qr-size);
  display: block;
}
</style>
