<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import bayouImg from '../assets/bayou_night.jpg'

const emit = defineEmits(['start', 'next'])

const volando = ref(false)
const reachedMoon = ref(false)
const fadingOut = ref(false)
const canvasRef = ref(null)

// 25 Luciérnagas de fondo con movimiento orgánico
const ambientFlies = ref([])

onMounted(() => {
  ambientFlies.value = Array.from({ length: 28 }, (_, i) => ({
    id: i,
    x: 5 + Math.random() * 90,
    y: 15 + Math.random() * 70,
    size: 2.5 + Math.random() * 3.5,
    dur: 3.2 + Math.random() * 4,
    del: -Math.random() * 6,
    driftX: -45 + Math.random() * 90,
    driftY: -35 + Math.random() * 70,
    color: Math.random() > 0.3 ? '#f6e47c' : '#bbf254',
  }))

  setupInteractiveCanvas()
})

// ── Sistema de partículas interactivas (cursor, toques y estelas) ──
let animFrameId = null
let particles = []

function setupInteractiveCanvas() {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')

  function resize() {
    canvas.width = window.innerWidth
    canvas.height = window.innerHeight
  }
  resize()
  window.addEventListener('resize', resize)

  function loop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height)

    for (let i = particles.length - 1; i >= 0; i--) {
      const p = particles[i]
      p.x += p.vx
      p.y += p.vy
      p.vx *= 0.98
      p.vy *= 0.98
      p.alpha -= p.decay
      p.size = Math.max(0, p.size - p.decay * 1.5)

      if (p.alpha <= 0) {
        particles.splice(i, 1)
        continue
      }

      ctx.save()
      ctx.globalAlpha = p.alpha
      ctx.shadowBlur = 10
      ctx.shadowColor = p.color
      ctx.fillStyle = p.color
      ctx.beginPath()
      ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2)
      ctx.fill()
      ctx.restore()
    }

    animFrameId = requestAnimationFrame(loop)
  }
  loop()

  onUnmounted(() => {
    window.removeEventListener('resize', resize)
    if (animFrameId) cancelAnimationFrame(animFrameId)
  })
}

function spawnSpark(x, y, count = 2, color = '#fbe886') {
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2
    const speed = 0.5 + Math.random() * 1.8
    particles.push({
      x: x + (Math.random() - 0.5) * 8,
      y: y + (Math.random() - 0.5) * 8,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed - 0.3,
      size: 2 + Math.random() * 2.5,
      alpha: 0.85 + Math.random() * 0.15,
      decay: 0.015 + Math.random() * 0.02,
      color,
    })
  }
}

function handlePointerMove(e) {
  if (volando.value) return
  const x = e.clientX || (e.touches && e.touches[0]?.clientX)
  const y = e.clientY || (e.touches && e.touches[0]?.clientY)
  if (x != null && y != null && Math.random() > 0.4) {
    spawnSpark(x, y, 1, Math.random() > 0.5 ? '#fbe886' : '#d2fa6a')
  }
}

// ── Iniciar vuelo hacia la luna al hacer click / tap ──
let trailInterval = null

function launch() {
  if (volando.value) return
  volando.value = true

  // Preparar historia de fondo
  emit('start')

  // Generar estela luminosa detrás de las 2 luciérnagas durante el vuelo
  const f1El = document.querySelector('.lover-fly.f1')
  const f2El = document.querySelector('.lover-fly.f2')

  trailInterval = setInterval(() => {
    if (!volando.value || reachedMoon.value) {
      clearInterval(trailInterval)
      return
    }
    if (f1El) {
      const r1 = f1El.getBoundingClientRect()
      spawnSpark(r1.left + r1.width / 2, r1.top + r1.height / 2, 2, '#fff194')
    }
    if (f2El) {
      const r2 = f2El.getBoundingClientRect()
      spawnSpark(r2.left + r2.width / 2, r2.top + r2.height / 2, 2, '#c8f06a')
    }
  }, 45)

  // 1. A los 2.3s llegan a la luna → explosión de luz mágica
  setTimeout(() => {
    reachedMoon.value = true
    clearInterval(trailInterval)
    // Destello de chispas en la luna
    const mx = window.innerWidth * 0.505
    const my = window.innerHeight * 0.165
    for (let i = 0; i < 35; i++) {
      spawnSpark(mx, my, 1, Math.random() > 0.4 ? '#fffce0' : '#ffd555')
    }
  }, 2350)

  // 2. A los 2.9s empieza la suave disolución cinematográfica
  setTimeout(() => {
    fadingOut.value = true
  }, 2900)

  // 3. A los 3.7s cambio completado a la primera escena
  setTimeout(() => {
    emit('next')
  }, 3700)
}
</script>

<template>
  <section
    class="intro-container"
    :class="{ 'fading-out': fadingOut }"
    @click="launch"
    @pointermove="handlePointerMove"
    @touchmove.passive="handlePointerMove"
    role="button"
    tabindex="0"
    aria-label="Toca la pantalla para comenzar"
    @keydown.enter="launch"
  >
    <!-- ══ Fondo fotográfico realista con efecto cámara de video (Ken Burns) ══ -->
    <div class="video-bg-layer">
      <img
        :src="bayouImg"
        alt="Noche en el pantano con luna alta y luciérnagas"
        class="bayou-photo"
      />
    </div>

    <!-- ══ Niebla suave y viviente sobre el agua del bayou ══ -->
    <div class="mist-layer mist-1" aria-hidden="true"></div>
    <div class="mist-layer mist-2" aria-hidden="true"></div>

    <!-- ══ Destello/Resplandor radiante en la luna alta ══ -->
    <div class="moon-radiance" aria-hidden="true"></div>

    <!-- ══ Reflejo palpitante en el lago ══ -->
    <div class="water-reflection-glow" aria-hidden="true"></div>

    <!-- ══ Canvas para chispas interactivas y estelas ══ -->
    <canvas ref="canvasRef" class="fx-canvas" aria-hidden="true"></canvas>

    <!-- ══ Luciérnagas de ambiente ══ -->
    <div class="ambient-swarm" aria-hidden="true">
      <span
        v-for="f in ambientFlies"
        :key="f.id"
        class="ambient-fly"
        :style="{
          left: f.x + '%',
          top: f.y + '%',
          '--fx': f.driftX + 'px',
          '--fy': f.driftY + 'px',
          '--dur': f.dur + 's',
          '--del': f.del + 's',
          '--sz': f.size + 'px',
          '--c': f.color,
        }"
      ></span>
    </div>

    <!-- ══ LAS DOS LUCIÉRNAGAS PROTAGONISTAS (Ray & Evangeline) ══ -->
    <div
      class="lover-fly f1"
      :class="{ soaring: volando, reached: reachedMoon }"
      aria-hidden="true"
    >
      <div class="wings"><i></i><i></i></div>
      <div class="fly-glow"></div>
      <div class="fly-nucleus"></div>
    </div>

    <div
      class="lover-fly f2"
      :class="{ soaring: volando, reached: reachedMoon }"
      aria-hidden="true"
    >
      <div class="wings"><i></i><i></i></div>
      <div class="fly-glow"></div>
      <div class="fly-nucleus"></div>
    </div>

    <!-- ══ Destello celestial cuando se tocan con la luna ══ -->
    <div
      class="moon-celestial-burst"
      :class="{ active: reachedMoon }"
      aria-hidden="true"
    ></div>

    <!-- ══ Textos e invitación interactiva ══ -->
    <div class="hero-content" :class="{ 'hero-hidden': volando }">
      <h1 class="main-quote">Agárrame de la mano,</h1>
      <p class="sub-quote">aquí empieza nuestra historia</p>

      <div class="touch-prompt">
        <span class="touch-glow"></span>
        <span class="touch-text">toca la pantalla</span>
        <span class="sparkle-icon">✦</span>
      </div>
    </div>
  </section>
</template>

<style scoped>
/* ──────────────────────────────────────────────────────────
   CONTENEDOR PRINCIPAL: Fullscreen & Interactivo
────────────────────────────────────────────────────────── */
.intro-container {
  position: fixed;
  inset: 0;
  z-index: 50;
  overflow: hidden;
  cursor: pointer;
  user-select: none;
  -webkit-user-select: none;
  outline: none;
  background: #060914;
  transition: opacity 0.85s cubic-bezier(0.4, 0, 0.2, 1), filter 0.85s ease;
}

.intro-container.fading-out {
  opacity: 0;
  filter: brightness(1.15);
  pointer-events: none;
}

/* ──────────────────────────────────────────────────────────
   FONDO REALISTA: Movimiento de cámara cinemático (estilo video)
────────────────────────────────────────────────────────── */
.video-bg-layer {
  position: absolute;
  inset: -4%;
  width: 108%;
  height: 108%;
  overflow: hidden;
  will-change: transform;
}

.bayou-photo {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: 50% 38%;
  display: block;
  /* Animación continua de video/cámara viva */
  animation: cinematicVideo 24s ease-in-out infinite alternate;
  transform-origin: 50.5% 25%;
}

@keyframes cinematicVideo {
  0% {
    transform: scale(1) translate3d(0, 0, 0);
  }
  50% {
    transform: scale(1.06) translate3d(-0.8%, -0.5%, 0);
  }
  100% {
    transform: scale(1.03) translate3d(0.7%, 0.4%, 0);
  }
}

/* ──────────────────────────────────────────────────────────
   ATMÓSFERA Y NIEBLA EN MOVIMIENTO
────────────────────────────────────────────────────────── */
.mist-layer {
  position: absolute;
  left: -20%;
  width: 140%;
  pointer-events: none;
  will-change: transform, opacity;
}

.mist-1 {
  top: 48%;
  height: 25%;
  background: radial-gradient(ellipse at 50% 50%, rgba(245, 175, 90, 0.12) 0%, rgba(100, 160, 200, 0.08) 50%, transparent 80%);
  filter: blur(20px);
  animation: mistDrift1 18s ease-in-out infinite alternate;
}

.mist-2 {
  top: 56%;
  height: 28%;
  background: radial-gradient(ellipse at 50% 50%, rgba(130, 185, 230, 0.1) 0%, rgba(40, 90, 120, 0.06) 60%, transparent 85%);
  filter: blur(28px);
  animation: mistDrift2 25s ease-in-out infinite alternate;
}

@keyframes mistDrift1 {
  0%   { transform: translateX(-4%) translateY(0); opacity: 0.6; }
  100% { transform: translateX(5%) translateY(-10px); opacity: 0.95; }
}

@keyframes mistDrift2 {
  0%   { transform: translateX(4%) translateY(5px); opacity: 0.5; }
  100% { transform: translateX(-5%) translateY(-5px); opacity: 0.85; }
}

/* ──────────────────────────────────────────────────────────
   RESPLANDOR DE LA LUNA ALTA & AGUA
────────────────────────────────────────────────────────── */
.moon-radiance {
  position: absolute;
  left: 50.5%;
  top: 16.5%;
  width: 260px;
  height: 260px;
  transform: translate(-50%, -50%);
  border-radius: 50%;
  background: radial-gradient(circle, rgba(255, 245, 180, 0.35) 0%, rgba(245, 210, 110, 0.15) 45%, transparent 72%);
  pointer-events: none;
  animation: moonPulse 5.5s ease-in-out infinite alternate;
}

@keyframes moonPulse {
  0%   { transform: translate(-50%, -50%) scale(0.95); opacity: 0.75; }
  100% { transform: translate(-50%, -50%) scale(1.15); opacity: 1; }
}

.water-reflection-glow {
  position: absolute;
  left: 50.5%;
  top: 82%;
  width: 220px;
  height: 120px;
  transform: translate(-50%, -50%);
  border-radius: 50%;
  background: radial-gradient(ellipse, rgba(255, 220, 130, 0.22) 0%, rgba(230, 170, 70, 0.08) 55%, transparent 75%);
  filter: blur(12px);
  pointer-events: none;
  animation: waterGlow 4.2s ease-in-out infinite alternate;
}

@keyframes waterGlow {
  0%   { transform: translate(-50%, -50%) scaleX(0.9) scaleY(0.85); opacity: 0.6; }
  100% { transform: translate(-50%, -50%) scaleX(1.15) scaleY(1.1);  opacity: 0.95; }
}

/* Canvas interactivo */
.fx-canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 10;
}

/* ──────────────────────────────────────────────────────────
   LUCIÉRNAGAS AMBIENTALES
────────────────────────────────────────────────────────── */
.ambient-swarm {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 4;
}

.ambient-fly {
  position: absolute;
  width: var(--sz);
  height: var(--sz);
  border-radius: 50%;
  background: var(--c);
  box-shadow:
    0 0 calc(var(--sz) * 1.6) calc(var(--sz) * 0.8) var(--c),
    0 0 calc(var(--sz) * 4) calc(var(--sz) * 1.5) rgba(240, 230, 120, 0.4);
  animation:
    flyDrift var(--dur) ease-in-out infinite alternate,
    flyBlink calc(var(--dur) * 0.6) ease-in-out infinite;
  animation-delay: var(--del);
  will-change: transform, opacity;
}

@keyframes flyDrift {
  0%   { transform: translate(0, 0); }
  100% { transform: translate(var(--fx), var(--fy)); }
}

@keyframes flyBlink {
  0%, 100% { opacity: 0.9; transform: scale(1); }
  35%      { opacity: 0.08; transform: scale(0.5); }
  52%      { opacity: 1; transform: scale(1.15); }
  70%      { opacity: 0.12; transform: scale(0.6); }
}

/* ──────────────────────────────────────────────────────────
   LAS DOS LUCIÉRNAGAS ENAMORADAS (Ray & Evangeline)
────────────────────────────────────────────────────────── */
.lover-fly {
  position: absolute;
  width: 22px;
  height: 22px;
  transform: translate(-50%, -50%);
  z-index: 20;
  pointer-events: none;
  transition: opacity 0.3s ease;
}

/* Posición inicial y baile cortejo */
.lover-fly.f1 {
  left: 36%;
  top: 64%;
  animation: loverDance1 4.5s ease-in-out infinite alternate;
}

.lover-fly.f2 {
  left: 64%;
  top: 61%;
  animation: loverDance2 5.2s ease-in-out infinite alternate;
}

@keyframes loverDance1 {
  0%   { transform: translate(-50%, -50%) translate3d(0, 0, 0) scale(1); }
  50%  { transform: translate(-50%, -50%) translate3d(18px, -24px, 0) scale(1.12); }
  100% { transform: translate(-50%, -50%) translate3d(-15px, 14px, 0) scale(0.95); }
}

@keyframes loverDance2 {
  0%   { transform: translate(-50%, -50%) translate3d(0, 0, 0) scale(0.98); }
  50%  { transform: translate(-50%, -50%) translate3d(-22px, -18px, 0) scale(1.15); }
  100% { transform: translate(-50%, -50%) translate3d(16px, 20px, 0) scale(1); }
}

/* ── Vuelo romántico hacia la luna alta (x: 50.5%, y: 16.5%) ── */
.lover-fly.f1.soaring {
  animation: flyToMoon1 2.4s cubic-bezier(0.42, 0.05, 0.32, 1) forwards !important;
}

.lover-fly.f2.soaring {
  animation: flyToMoon2 2.4s cubic-bezier(0.38, 0.02, 0.28, 1) 0.08s forwards !important;
}

@keyframes flyToMoon1 {
  0% {
    left: 36%;
    top: 64%;
    transform: translate(-50%, -50%) scale(1);
    opacity: 1;
  }
  35% {
    left: 28%;
    top: 45%;
    transform: translate(-50%, -50%) scale(1.35);
    opacity: 1;
  }
  70% {
    left: 44%;
    top: 26%;
    transform: translate(-50%, -50%) scale(1.1);
    opacity: 1;
  }
  100% {
    left: 50.5%;
    top: 16.5%;
    transform: translate(-50%, -50%) scale(0.3);
    opacity: 0;
  }
}

@keyframes flyToMoon2 {
  0% {
    left: 64%;
    top: 61%;
    transform: translate(-50%, -50%) scale(1);
    opacity: 1;
  }
  35% {
    left: 71%;
    top: 42%;
    transform: translate(-50%, -50%) scale(1.3);
    opacity: 1;
  }
  70% {
    left: 56%;
    top: 25%;
    transform: translate(-50%, -50%) scale(1.05);
    opacity: 1;
  }
  100% {
    left: 50.5%;
    top: 16.5%;
    transform: translate(-50%, -50%) scale(0.3);
    opacity: 0;
  }
}

.lover-fly.reached {
  opacity: 0;
}

/* Núcleo y halo luminoso de las luciérnagas protagonistas */
.fly-nucleus {
  position: absolute;
  inset: 4px;
  border-radius: 50%;
  background: radial-gradient(circle at 35% 35%, #ffffff 0%, #fff7a0 45%, #bdf740 100%);
  box-shadow:
    0 0 10px 4px #e4ff68,
    0 0 24px 10px rgba(210, 255, 80, 0.75),
    0 0 50px 22px rgba(180, 245, 60, 0.35);
  animation: nucleusBeat 1.4s ease-in-out infinite alternate;
}

.fly-glow {
  position: absolute;
  inset: -14px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(240, 255, 140, 0.28) 0%, transparent 72%);
  animation: glowPulse 1.8s ease-in-out infinite alternate;
}

/* Alitas en aleteo rápido */
.wings {
  position: absolute;
  top: -3px;
  left: 50%;
  transform: translateX(-50%);
  width: 16px;
  height: 8px;
}

.wings i {
  position: absolute;
  top: 0;
  width: 7px;
  height: 8px;
  border-radius: 50% 50% 20% 20%;
  background: rgba(255, 255, 255, 0.55);
  transform-origin: bottom center;
}

.wings i:first-child {
  left: 0;
  animation: flapLeft 0.12s ease-in-out infinite alternate;
}

.wings i:last-child {
  right: 0;
  animation: flapRight 0.12s ease-in-out infinite alternate;
}

@keyframes flapLeft {
  0%   { transform: rotate(-35deg) scaleY(0.7); }
  100% { transform: rotate(15deg) scaleY(1.1); }
}

@keyframes flapRight {
  0%   { transform: rotate(35deg) scaleY(0.7); }
  100% { transform: rotate(-15deg) scaleY(1.1); }
}

@keyframes nucleusBeat {
  0%   { transform: scale(0.92); filter: brightness(0.95); }
  100% { transform: scale(1.18); filter: brightness(1.2); }
}

@keyframes glowPulse {
  0%   { transform: scale(0.8); opacity: 0.6; }
  100% { transform: scale(1.35); opacity: 1; }
}

/* ──────────────────────────────────────────────────────────
   DESTELO CELESTIAL EN LA LUNA (Al llegar las luciérnagas)
────────────────────────────────────────────────────────── */
.moon-celestial-burst {
  position: absolute;
  left: 50.5%;
  top: 16.5%;
  width: 0;
  height: 0;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  background: radial-gradient(circle, rgba(255, 255, 230, 1) 0%, rgba(255, 225, 120, 0.85) 30%, rgba(240, 180, 50, 0.4) 60%, transparent 80%);
  pointer-events: none;
  z-index: 25;
  opacity: 0;
}

.moon-celestial-burst.active {
  animation: burstFlare 1.1s cubic-bezier(0.1, 0.8, 0.2, 1) forwards;
}

@keyframes burstFlare {
  0% {
    width: 20px;
    height: 20px;
    opacity: 0;
    box-shadow: 0 0 0px 0px rgba(255, 240, 180, 0.8);
  }
  40% {
    width: 380px;
    height: 380px;
    opacity: 1;
    box-shadow:
      0 0 80px 40px rgba(255, 240, 180, 0.95),
      0 0 160px 80px rgba(255, 200, 80, 0.5);
  }
  100% {
    width: 650px;
    height: 650px;
    opacity: 0;
    box-shadow: 0 0 200px 100px rgba(255, 210, 90, 0);
  }
}

/* ──────────────────────────────────────────────────────────
   TEXTOS Y BOTÓN DE INVITACIÓN
────────────────────────────────────────────────────────── */
.hero-content {
  position: absolute;
  bottom: 11vh;
  left: 50%;
  transform: translateX(-50%);
  text-align: center;
  z-index: 30;
  pointer-events: none;
  white-space: nowrap;
  transition: opacity 0.55s ease, transform 0.55s ease;
}

.hero-content.hero-hidden {
  opacity: 0;
  transform: translateX(-50%) translateY(20px);
}

.main-quote {
  margin: 0 0 8px;
  font-family: 'Great Vibes', cursive;
  font-size: clamp(2.4rem, 6.2vw, 4.2rem);
  font-weight: 400;
  color: #fff8e2;
  text-shadow:
    0 0 25px rgba(255, 235, 160, 0.9),
    0 0 60px rgba(240, 200, 90, 0.5),
    0 3px 12px rgba(0, 0, 0, 0.85);
  animation: titleShine 4s ease-in-out infinite alternate;
}

@keyframes titleShine {
  0%   { text-shadow: 0 0 25px rgba(255, 235, 160, 0.85), 0 0 50px rgba(240, 200, 90, 0.4), 0 3px 12px rgba(0, 0, 0, 0.85); }
  100% { text-shadow: 0 0 40px rgba(255, 245, 190, 1),    0 0 85px rgba(255, 210, 100, 0.65), 0 3px 12px rgba(0, 0, 0, 0.85); }
}

.sub-quote {
  margin: 0 0 28px;
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: clamp(1.05rem, 2.9vw, 1.6rem);
  font-style: italic;
  letter-spacing: 0.06em;
  color: rgba(255, 245, 215, 0.92);
  text-shadow:
    0 2px 10px rgba(0, 0, 0, 0.9),
    0 0 20px rgba(230, 190, 110, 0.3);
}

/* Botón interactivo "toca la pantalla" */
.touch-prompt {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 12px 34px;
  border-radius: 999px;
  background: rgba(8, 14, 28, 0.45);
  border: 1px solid rgba(255, 230, 150, 0.55);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  box-shadow:
    0 8px 30px rgba(0, 0, 0, 0.55),
    inset 0 1px 1px rgba(255, 255, 255, 0.25),
    0 0 25px rgba(245, 220, 120, 0.25);
  cursor: pointer;
  pointer-events: auto;
  transition: transform 0.3s ease, box-shadow 0.3s ease, border-color 0.3s ease;
  animation: promptFloat 3s ease-in-out infinite;
}

.touch-prompt:hover {
  transform: translateY(-2px) scale(1.04);
  border-color: rgba(255, 240, 180, 0.9);
  box-shadow:
    0 12px 35px rgba(0, 0, 0, 0.65),
    0 0 35px rgba(255, 230, 130, 0.5);
}

.touch-text {
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: clamp(1rem, 2.4vw, 1.15rem);
  font-style: italic;
  letter-spacing: 0.08em;
  color: #fff9e6;
  text-shadow: 0 1px 6px rgba(0, 0, 0, 0.8);
}

.sparkle-icon {
  font-size: 1.1rem;
  color: #ffe885;
  display: inline-block;
  animation: spinSparkle 3.5s linear infinite;
  text-shadow: 0 0 10px #ffe885;
}

@keyframes spinSparkle {
  0%   { transform: rotate(0deg) scale(1); }
  50%  { transform: rotate(180deg) scale(1.35); }
  100% { transform: rotate(360deg) scale(1); }
}

@keyframes promptFloat {
  0%, 100% { transform: translateY(0) scale(1); }
  50%      { transform: translateY(-5px) scale(1.02); }
}

@media (max-width: 600px) {
  .hero-content {
    bottom: 8vh;
    white-space: normal;
    padding: 0 20px;
    width: 100%;
  }
  .touch-prompt {
    padding: 10px 26px;
  }
}
</style>
