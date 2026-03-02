<script setup lang="ts">
import { computed, ref } from 'vue'

type Step = 1 | 2 | 3

const props = withDefaults(
  defineProps<{
    activeStep?: Step
  }>(),
  {},
)

const localStep = ref<Step>(1)
const localBadge = ref(false)

const currentStep = computed<Step>(() => props.activeStep ?? localStep.value)
const showBadge = computed(() => (props.activeStep ? props.activeStep === 3 : localBadge.value))

function activate(step: Step) {
  if (props.activeStep) return
  localStep.value = step
}

function toggleBadge() {
  if (props.activeStep) return
  localBadge.value = !localBadge.value
}
</script>

<template>
  <section class="rpi" aria-label="Research Plan Implement workflow">
    <div class="controls" role="tablist" aria-label="RPI step controls">
      <button class="chip" :class="{ active: currentStep === 1 }" type="button" @click="activate(1)">
        Research
      </button>
      <button class="chip" :class="{ active: currentStep === 2 }" type="button" @click="activate(2)">
        Plan
      </button>
      <button class="chip" :class="{ active: currentStep === 3 }" type="button" @click="activate(3)">
        Implement
      </button>
      <button class="chip accent" :class="{ active: showBadge }" type="button" @click="toggleBadge()">
        Drift rule
      </button>
    </div>

    <div class="lane">
      <article class="arrow" :class="{ on: currentStep >= 1 }">
        <h4>1. Research</h4>
        <p>Compress truth</p>
      </article>

      <article class="arrow" :class="{ on: currentStep >= 2 }">
        <h4>2. Plan</h4>
        <p>Compress intent</p>
      </article>

      <article class="arrow" :class="{ on: currentStep >= 3 }">
        <h4>3. Implement</h4>
        <p>Clean execution</p>
      </article>
    </div>

    <p class="badge" :class="{ show: showBadge }">Compact + restart when drift appears.</p>
  </section>
</template>

<style scoped>
.rpi {
  display: grid;
  gap: 0.34rem;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.3rem;
}

.chip {
  border-radius: 999px;
  border: 2px solid #9db1c5;
  background: #e8f1f9;
  color: #1f2937;
  padding: 0.1rem 0.4rem;
  font-size: 0.54rem;
  font-weight: 800;
  cursor: pointer;
}

.chip.active {
  border-color: #2563eb;
  background: #dbeafe;
}

.chip.accent.active {
  border-color: #16a34a;
  background: #dcfce7;
}

.lane {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 0.22rem;
}

.arrow {
  position: relative;
  border-radius: 10px;
  border: 2px solid #8fa4b8;
  background: linear-gradient(135deg, #dce9f7, #b7d0ec);
  min-height: 2.7rem;
  padding: 0.35rem 0.9rem 0.35rem 0.45rem;
  opacity: 0.45;
}

.arrow::after {
  content: '';
  position: absolute;
  top: 50%;
  right: -0.65rem;
  transform: translateY(-50%);
  border-top: 0.72rem solid transparent;
  border-bottom: 0.72rem solid transparent;
  border-left: 0.65rem solid #9db8d4;
}

.arrow:last-child::after {
  display: none;
}

.arrow.on {
  opacity: 1;
  border-color: #2563eb;
}

.arrow h4 {
  margin: 0;
  font-size: 0.72rem;
  font-weight: 900;
  color: #0f172a;
}

.arrow p {
  margin: 0.05rem 0 0;
  font-size: 0.56rem;
  color: #334155;
}

.badge {
  justify-self: end;
  margin: 0;
  font-size: 0.56rem;
  font-weight: 800;
  color: #166534;
  border: 2px solid #86efac;
  background: #f0fdf4;
  border-radius: 999px;
  padding: 0.1rem 0.45rem;
  opacity: 0;
}

.badge.show {
  opacity: 1;
}
</style>
