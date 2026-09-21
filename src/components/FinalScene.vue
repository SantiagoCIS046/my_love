<script setup>
import { ref, onBeforeUnmount } from 'vue'
import confetti from 'canvas-confetti'
import { cartaFinal, firmaCarta } from '../data/content'
import { config } from '../data/config'

const etapa = ref('entrada') // entrada -> carta -> final
const heartShape = confetti.shapeFromPath({
  path: 'M167 72c19,-38 37,-56 75,-56 42,0 76,33 76,75 0,76 -76,151 -151,227 -76,-76 -151,-151 -151,-227 0,-42 33,-75 75,-75 38,0 57,18 76,56z',
})

const colors = ['#c9556e', '#e8a4b4', '#c9a24b', '#fdf6ee', '#a92440']
let fireworksTimer = null

const stars = Array.from({ length: 60 }, (_, s) => {
  const r = ((s * 9301 + 49297) % 233280) / 233280
  const r2 = ((s * 4801 + 1723) % 10000) / 10000
  return {
    left: (r * 100).toFixed(2) + '%',
    top: (r2 * 55).toFixed(2) + '%',
    animationDelay: (r2 * 4).toFixed(2) + 's',
    animationDuration: (2 + r * 3).toFixed(2) + 's',
  }
})

function abrirCarta() {
  etapa.value = 'carta'
}

function granFinal() {
  etapa.value = 'final'
  iniciarFuegos()
}

function burst(x, y) {
  confetti({
    particleCount: 40,
    spread: 70,
    startVelocity: 38,
    origin: { x, y },
    colors,
    shapes: [heartShape],
    scalar: 1.6,
    ticks: 220,
    gravity: 0.9,
    disableForReducedMotion: true,
  })
  confetti({
    particleCount: 22,
    spread: 100,
    startVelocity: 30,
    origin: { x, y },
    colors,
    shapes: ['circle'],
    scalar: 0.9,
    ticks: 200,
    gravity: 0.9,
    disableForReducedMotion: true,
  })
}

function iniciarFuegos() {
  const raf = () => {
    burst(0.15 + Math.random() * 0.7, 0.2 + Math.random() * 0.35)
  }
  raf()
  fireworksTimer = setInterval(() => {
    raf()
    if (Math.random() > 0.5) raf()
  }, 900)
}

onBeforeUnmount(() => {
  if (fireworksTimer) clearInterval(fireworksTimer)
})
</script>

<template>
  <!-- ════════ Etapa 1: invitación ════════ -->
  <section v-if="etapa === 'entrada'" class="scene scene-center">
    <div class="fade-up">
      <div class="big-heart">💗</div>
      <h1 class="script-title">Guardé lo mejor para el final…</h1>
      <button class="btn-love" @click="abrirCarta">Abrir mi corazón&nbsp;💌</button>
    </div>
  </section>

  <!-- ════════ Etapa 2: la carta ════════ -->
  <section v-else-if="etapa === 'carta'" class="scene scene-center">
    <div class="paper letter fade-up">
      <div class="ornament">❦</div>
      <p v-for="(par, i) in cartaFinal" :key="i" class="paragraph" :class="{ saludo: i === 0 }">
        {{ par }}
      </p>
      <div class="firma-block">
        <p class="firma script-title">{{ firmaCarta.despedida }}</p>
        <p class="firma-nombre">{{ firmaCarta.autor }}</p>
      </div>
      <div class="letter-actions">
        <button class="btn-love" @click="granFinal">Y ahora, el gran final…&nbsp;✨</button>
      </div>
    </div>
  </section>

  <!-- ════════ Etapa 3: final estilo Disney ════════ -->
  <section v-else class="scene disney">
    <div class="stars" aria-hidden="true">
      <span v-for="(s, i) in stars" :key="i" class="star" :style="s"></span>
    </div>
    <div class="moon" aria-hidden="true"></div>
    <div class="castle" aria-hidden="true">
      <div class="tower t1"></div>
      <div class="tower t2"></div>
      <div class="tower t3"></div>
      <div class="base"></div>
    </div>

    <div class="finale-text">
      <p class="finale-script">Y así, después de tanto amor…</p>
      <h1 class="finale-title">Y vivieron felices<br />para siempre</h1>
      <p class="finale-sub">El fin… de esta carta. Nunca de nosotros. ♾️</p>
    </div>
  </section>
</template>

<style scoped>
.scene {
  position: relative;
  z-index: 1;
  min-height: 100svh;
  padding: 48px 20px;
}

.scene-center {
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.big-heart {
  font-size: clamp(4rem, 12vw, 6.5rem);
  margin-bottom: 12px;
  animation: heartbeat 2.4s ease-in-out infinite;
}

h1 {
  margin: 0 0 30px;
  font-size: clamp(2.2rem, 6vw, 3.4rem);
}

/* ---------- Carta ---------- */
.letter {
  width: min(640px, 100%);
  margin: 0 auto;
  padding: clamp(30px, 5vw, 56px);
  text-align: left;
}

.ornament {
  text-align: center;
  font-size: 1.6rem;
  color: var(--gold);
  margin-bottom: 18px;
}

.paragraph {
  margin: 0 0 20px;
  font-size: clamp(1.05rem, 2.8vw, 1.2rem);
  line-height: 1.65;
  color: var(--ink);
}

.paragraph.saludo {
  font-family: var(--font-script);
  font-size: clamp(1.5rem, 4vw, 1.9rem);
  color: var(--wine);
}

.firma-block {
  margin: 32px 0 0;
  text-align: right;
}

.firma {
  margin: 0;
  text-align: right;
  font-size: clamp(1.6rem, 4.5vw, 2.1rem);
  color: var(--wine);
}

.firma-nombre {
  margin: 8px 0 0;
  text-align: right;
  font-family: var(--font-script);
  font-size: clamp(1.25rem, 3.6vw, 1.65rem);
  color: var(--wine);
  line-height: 1.35;
}

.letter-actions {
  margin-top: 34px;
  text-align: center;
}

/* ---------- Final Disney ---------- */
.disney {
  display: flex;
  align-items: center;
  justify-content: center;
  background:
    radial-gradient(ellipse at 50% 115%, rgba(201, 85, 110, 0.35) 0%, transparent 55%),
    linear-gradient(180deg, #0d1030 0%, #1a1440 45%, #2c1240 75%, #4a1235 100%);
  overflow: hidden;
}

.finale-text {
  position: relative;
  z-index: 5;
  text-align: center;
  color: #fdf6ee;
  animation: fadeUp 2.2s cubic-bezier(0.22, 1, 0.36, 1) 0.6s both;
}

.finale-script {
  margin: 0 0 18px;
  font-family: var(--font-script);
  font-size: clamp(1.4rem, 4vw, 2rem);
  color: #e8a4b4;
}

.finale-title {
  margin: 0;
  font-family: var(--font-title);
  font-weight: 400;
  font-size: clamp(2.6rem, 8vw, 5rem);
  line-height: 1.15;
  color: #fdf6ee;
  text-shadow:
    0 0 24px rgba(232, 164, 180, 0.55),
    0 4px 30px rgba(0, 0, 0, 0.5);
}

.finale-sub {
  margin: 26px 0 0;
  font-style: italic;
  font-size: clamp(1rem, 3vw, 1.25rem);
  color: rgba(253, 246, 238, 0.75);
}

/* Luna */
.moon {
  position: absolute;
  top: 9%;
  right: 12%;
  width: clamp(70px, 10vw, 110px);
  aspect-ratio: 1;
  border-radius: 50%;
  background: radial-gradient(circle at 38% 35%, #fff9e8 0%, #f5e6b8 55%, #e8d193 100%);
  box-shadow:
    0 0 40px rgba(255, 244, 200, 0.5),
    0 0 120px rgba(255, 244, 200, 0.25);
  animation: fadeUp 2s ease 0.3s both;
}

/* Estrellas */
.stars {
  position: absolute;
  inset: 0;
}

.star {
  position: absolute;
  width: 3px;
  height: 3px;
  border-radius: 50%;
  background: #fff;
  animation: twinkle linear infinite;
}

@keyframes twinkle {
  0%, 100% { opacity: 0.15; transform: scale(1); }
  50% { opacity: 1; transform: scale(1.5); }
}

/* Castillo (silueta) */
.castle {
  position: absolute;
  bottom: -6px;
  left: 50%;
  transform: translateX(-50%);
  width: min(520px, 90vw);
  height: 34vh;
  opacity: 0.92;
  filter: drop-shadow(0 -4px 30px rgba(201, 85, 110, 0.35));
}

.tower {
  position: absolute;
  bottom: 0;
  background: linear-gradient(180deg, #1d1030 0%, #120a22 100%);
  clip-path: polygon(50% 0%, 68% 18%, 60% 18%, 60% 100%, 40% 100%, 40% 18%, 32% 18%);
}

.tower::after {
  content: '';
  position: absolute;
  top: 16%;
  left: 46%;
  width: 8%;
  height: 10%;
  border-radius: 50%;
  background: #f5e6b8;
  box-shadow: 0 0 14px rgba(255, 244, 200, 0.8);
}

.t1 { left: 4%;  width: 24%; height: 62%; }
.t2 { left: 38%; width: 24%; height: 100%; }
.t3 { right: 4%; width: 24%; height: 74%; }

.base {
  position: absolute;
  bottom: 0;
  left: 10%;
  right: 10%;
  height: 40%;
  background: linear-gradient(180deg, #1d1030 0%, #0f0820 100%);
  clip-path: polygon(0 100%, 0 35%, 12% 0, 25% 30%, 38% 5%, 50% 32%, 62% 5%, 75% 30%, 88% 0, 100% 35%, 100% 100%);
}

/* ---------- Móvil ---------- */
@media (max-width: 640px) {
  .scene {
    padding: 24px 14px;
  }

  .letter {
    padding: 26px 20px;
  }

  .paragraph {
    font-size: 1.02rem;
    margin-bottom: 16px;
    line-height: 1.6;
  }

  .letter-actions .btn-love {
    width: 100%;
    justify-content: center;
    font-size: 1.02rem;
    padding: 13px 20px;
  }

  .moon {
    top: 7%;
    right: 8%;
  }

  .castle {
    width: 100vw;
    height: 26vh;
  }

  .finale-title {
    font-size: clamp(2.1rem, 10vw, 3rem);
  }

  .finale-sub {
    padding: 0 12px;
  }
}

@media (max-height: 620px) {
  .castle {
    height: 20vh;
  }
}
</style>
