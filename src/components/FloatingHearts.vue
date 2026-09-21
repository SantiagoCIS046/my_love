<script setup>
import { ref, onMounted } from 'vue'

const hearts = ref([])

onMounted(() => {
  hearts.value = Array.from({ length: 16 }, (_, i) => ({
    id: i,
    left: Math.random() * 100,
    size: 12 + Math.random() * 22,
    duration: 9 + Math.random() * 12,
    delay: -Math.random() * 18,
    opacity: 0.25 + Math.random() * 0.4,
    drift: -30 + Math.random() * 60,
    char: Math.random() > 0.5 ? '♥' : '❤',
  }))
})
</script>

<template>
  <div class="floating-hearts" aria-hidden="true">
    <span
      v-for="h in hearts"
      :key="h.id"
      class="heart"
      :style="{
        left: h.left + '%',
        '--size': h.size + 'px',
        '--duration': h.duration + 's',
        '--delay': h.delay + 's',
        '--opacity': h.opacity,
        '--drift': h.drift + 'px',
      }"
    >
      {{ h.char }}
    </span>
  </div>
</template>

<style scoped>
.floating-hearts {
  position: fixed;
  inset: 0;
  overflow: hidden;
  pointer-events: none;
  z-index: 0;
}

.heart {
  position: absolute;
  bottom: -8vh;
  font-size: var(--size);
  color: var(--rose);
  opacity: var(--opacity);
  animation: floatUp var(--duration) linear infinite;
  animation-delay: var(--delay);
  will-change: transform;
}

@keyframes floatUp {
  0% {
    transform: translateY(0) translateX(0) rotate(0deg);
    opacity: 0;
  }
  8% {
    opacity: var(--opacity);
  }
  92% {
    opacity: var(--opacity);
  }
  100% {
    transform: translateY(-115vh) translateX(var(--drift)) rotate(40deg);
    opacity: 0;
  }
}
</style>
