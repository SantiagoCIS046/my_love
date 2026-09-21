<script setup>
import { ref, computed, onMounted } from 'vue'
import { config } from '../data/config'

const emit = defineEmits(['next'])

const opened = ref(false)
const leaving = ref(false)

const tiempo = computed(() => {
  const start = new Date(config.anniversaryDate + 'T00:00:00')
  const now = new Date()
  let years = now.getFullYear() - start.getFullYear()
  let months = now.getMonth() - start.getMonth()
  let days = now.getDate() - start.getDate()
  if (days < 0) {
    months--
    days += new Date(now.getFullYear(), now.getMonth(), 0).getDate()
  }
  if (months < 0) {
    years--
    months += 12
  }
  const totalDays = Math.floor((now - start) / 86400000)
  return { years, months, days, totalDays }
})

function abrir() {
  if (opened.value) return
  opened.value = true
  // Tiempo para que se vea la animación del sobre antes de cambiar de escena
  setTimeout(() => (leaving.value = true), 1600)
  setTimeout(() => emit('next'), 2400)
}

onMounted(() => {})
</script>

<template>
  <section class="scene" :class="{ leaving }">
    <p class="hint fade-up">{{ config.envelopeHint }}</p>

    <!-- Contador de años -->
    <div class="counter fade-up" style="animation-delay: 0.3s">
      <span class="big-number">{{ tiempo.years }}</span>
      <span class="years-label">años juntos</span>
      <span class="detail">
        {{ tiempo.totalDays.toLocaleString('es') }} días ·
        {{ tiempo.months }} meses y {{ tiempo.days }} días robándome el corazón
      </span>
    </div>

    <!-- Sobre -->
    <div
      class="envelope-wrap fade-up"
      style="animation-delay: 0.6s"
      :class="{ opened }"
    >
      <button class="envelope" :aria-label="'Abrir la carta'" @click="abrir">
        <div class="env-back"></div>
        <div class="env-letter">
          <span class="mini-heart">♥</span>
        </div>
        <div class="env-pocket"></div>
        <div class="env-flap">
          <span class="wax-seal">♥</span>
        </div>
      </button>
      <p class="tap-hint">toca el sobre</p>
    </div>
  </section>
</template>

<style scoped>
.scene {
  position: relative;
  z-index: 1;
  min-height: 100svh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 34px;
  padding: 40px 20px;
  transition: opacity 0.8s ease, transform 0.8s ease;
}

.scene.leaving {
  opacity: 0;
  transform: scale(1.08);
}

.hint {
  margin: 0;
  font-family: var(--font-script);
  font-size: clamp(1.5rem, 4vw, 2.2rem);
  color: var(--wine);
  opacity: 0.85;
}

/* ---------- Contador ---------- */
.counter {
  display: flex;
  flex-direction: column;
  align-items: center;
  line-height: 1.1;
}

.big-number {
  font-family: var(--font-serif);
  font-weight: 600;
  font-size: clamp(6rem, 20vw, 11rem);
  background: linear-gradient(160deg, var(--wine) 20%, var(--rose) 80%);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  text-shadow: 0 20px 40px rgba(109, 26, 45, 0.12);
  animation: heartbeat 2.6s ease-in-out infinite;
}

.years-label {
  font-family: var(--font-script);
  font-size: clamp(1.8rem, 5vw, 2.6rem);
  color: var(--wine);
  margin-top: -10px;
}

.detail {
  margin-top: 12px;
  font-style: italic;
  font-size: clamp(0.95rem, 2.5vw, 1.1rem);
  color: var(--ink);
  opacity: 0.75;
  letter-spacing: 0.03em;
}

/* ---------- Sobre ---------- */
.envelope-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 18px;
}

.envelope {
  position: relative;
  width: min(340px, 84vw);
  aspect-ratio: 340 / 225;
  background: none;
  border: none;
  padding: 0;
  perspective: 900px;
  filter: drop-shadow(0 18px 35px rgba(74, 15, 30, 0.3));
  transition: transform 0.4s ease;
}

.envelope:hover {
  transform: translateY(-6px) rotate(-1deg);
}

.env-back {
  position: absolute;
  inset: 0;
  background: linear-gradient(160deg, #f3d9c9 0%, #ecc4ae 100%);
  border-radius: 8px;
}

.env-letter {
  position: absolute;
  left: 6%;
  right: 6%;
  top: 8%;
  height: 84%;
  background: var(--paper);
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 8px rgba(74, 15, 30, 0.15);
  transition: transform 1s cubic-bezier(0.22, 1, 0.36, 1) 0.55s;
  z-index: 1;
}

.mini-heart {
  font-size: 34px;
  color: var(--rose);
  animation: heartbeat 2s ease-in-out infinite;
}

.env-pocket {
  position: absolute;
  inset: 0;
  background: linear-gradient(160deg, #efd0bc 0%, #e5b89a 100%);
  clip-path: polygon(0 0, 50% 55%, 100% 0, 100% 100%, 0 100%);
  border-radius: 8px;
  z-index: 2;
}

.env-flap {
  position: absolute;
  inset: 0;
  background: linear-gradient(160deg, #e9c3a4 0%, #dda87f 100%);
  clip-path: polygon(0 0, 100% 0, 50% 58%);
  border-radius: 8px 8px 0 0;
  transform-origin: top center;
  transform: rotateX(0deg);
  transition: transform 0.9s cubic-bezier(0.65, 0, 0.35, 1);
  z-index: 3;
  backface-visibility: visible;
}

.wax-seal {
  position: absolute;
  left: 50%;
  top: 42%;
  transform: translate(-50%, -50%);
  width: 58px;
  height: 58px;
  display: grid;
  place-items: center;
  font-size: 24px;
  color: #ffd9de;
  background: radial-gradient(circle at 35% 30%, #a92440, #6d1a2d 70%);
  border-radius: 50%;
  box-shadow:
    0 4px 10px rgba(74, 15, 30, 0.4),
    inset 0 2px 4px rgba(255, 255, 255, 0.25);
  transition: opacity 0.4s ease;
}

/* Estado abierto */
.opened .env-flap {
  transform: rotateX(178deg);
  z-index: 0;
}

.opened .wax-seal {
  opacity: 0;
}

.opened .env-letter {
  transform: translateY(-56%) scale(1.06);
  z-index: 2;
}

.opened .envelope {
  transform: translateY(4px);
}

.tap-hint {
  margin: 0;
  font-style: italic;
  font-size: 1rem;
  color: var(--wine);
  opacity: 0.6;
  transition: opacity 0.4s ease;
}

.opened .tap-hint {
  opacity: 0;
}

@media (max-width: 480px), (max-height: 700px) {
  .scene {
    gap: 22px;
    padding: 24px 16px 72px;
  }

  .detail {
    max-width: 30ch;
    text-align: center;
  }

  .wax-seal {
    width: 52px;
    height: 52px;
    font-size: 21px;
  }
}
</style>
