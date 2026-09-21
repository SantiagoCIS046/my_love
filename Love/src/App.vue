<script setup>
import { ref, computed } from 'vue'
import FloatingHearts from './components/FloatingHearts.vue'
import EnvelopeScene from './components/EnvelopeScene.vue'
import PhotoCarousel from './components/PhotoCarousel.vue'
import LoveContract from './components/LoveContract.vue'
import MemoriesScene from './components/MemoriesScene.vue'
import FinalScene from './components/FinalScene.vue'

const scenes = [
  EnvelopeScene,
  PhotoCarousel,
  LoveContract,
  MemoriesScene,
  FinalScene,
]

const current = ref(0)
const isFinal = computed(() => current.value === scenes.length - 1)

function next() {
  if (current.value < scenes.length - 1) current.value++
}

function goTo(i) {
  current.value = i
}
</script>

<template>
  <FloatingHearts v-if="!isFinal" />

  <Transition name="scene" mode="out-in">
    <component :is="scenes[current]" :key="current" @next="next" />
  </Transition>

  <!-- Progreso: un corazón por escena -->
  <nav v-if="!isFinal" class="progress" aria-label="Progreso">
    <button
      v-for="(s, i) in scenes"
      :key="i"
      class="dot"
      :class="{ active: i === current, done: i < current }"
      :aria-label="'Ir a la escena ' + (i + 1)"
      @click="goTo(i)"
    >
      ♥
    </button>
  </nav>
</template>

<style scoped>
.progress {
  position: fixed;
  bottom: calc(18px + env(safe-area-inset-bottom, 0px));
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 14px;
  z-index: 10;
  background: rgba(255, 250, 243, 0.75);
  backdrop-filter: blur(8px);
  padding: 8px 18px;
  border-radius: 999px;
  border: 1px solid rgba(109, 26, 45, 0.12);
  box-shadow: 0 6px 20px rgba(74, 15, 30, 0.15);
}

.dot {
  background: none;
  border: none;
  padding: 0;
  font-size: 15px;
  line-height: 1;
  color: rgba(109, 26, 45, 0.25);
  transition: color 0.3s ease, transform 0.3s ease;
}

.dot.done {
  color: var(--rose-soft);
}

.dot.active {
  color: var(--wine);
  transform: scale(1.45);
  animation: heartbeat 2s ease-in-out infinite;
}

.dot:hover {
  transform: scale(1.3);
}

@media (max-width: 640px) {
  .progress {
    gap: 12px;
    padding: 7px 15px;
    bottom: calc(12px + env(safe-area-inset-bottom, 0px));
  }

  .dot {
    font-size: 13px;
  }
}
</style>
