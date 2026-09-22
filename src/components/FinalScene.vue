<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import confetti from 'canvas-confetti'
import { cartaFinal, firmaCarta } from '../data/content'
import disneyCastleImg from '../assets/disney_castle.jpg'

const emit = defineEmits(['next', 'restart'])
const etapa = ref('entrada') // entrada -> carta -> final
const arcCanvasRef = ref(null)
const starActive = ref(false)

function reiniciar() {
  emit('restart')
}

const heartShape = confetti.shapeFromPath({
  path: 'M167 72c19,-38 37,-56 75,-56 42,0 76,33 76,75 0,76 -76,151 -151,227 -76,-76 -151,-151 -151,-227 0,-42 33,-75 75,-75 38,0 57,18 76,56z',
})

const colors = ['#fde047', '#f472b6', '#67e8f9', '#ffffff', '#e879f9', '#fbbf24']
let fireworksTimer = null
let starCycleTimer = null
let animId = null
let starParticles = []

function abrirCarta() {
  etapa.value = 'carta'
}

function granFinal() {
  etapa.value = 'final'
  // Lanzar la intro cinemática de la estrella de Disney
  setTimeout(() => {
    lanzarEstrellaDisney()
    iniciarFuegosDisney()
  }, 400)
}

function lanzarEstrellaDisney() {
  starActive.value = true
  initStardustCanvas()
}

function initStardustCanvas() {
  const canvas = arcCanvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')

  function resize() {
    canvas.width = window.innerWidth
    canvas.height = window.innerHeight
  }
  resize()

  const startTime = performance.now()
  const duration = 3400 // 3.4s para completar el arco majestuoso

  // Parábola icónica de Disney sobre la aguja del castillo
  function getArcPos(t) {
    const w = canvas.width
    const h = canvas.height
    // Inicio: 12% ancho, 64% alto | Cima del arco: 50% ancho, 11% alto | Fin: 88% ancho, 62% alto
    const p0 = { x: w * 0.12, y: h * 0.64 }
    const p1 = { x: w * 0.32, y: h * 0.08 }
    const p2 = { x: w * 0.68, y: h * 0.08 }
    const p3 = { x: w * 0.88, y: h * 0.62 }

    const cx = 3 * (p1.x - p0.x)
    const bx = 3 * (p2.x - p1.x) - cx
    const ax = p3.x - p0.x - cx - bx

    const cy = 3 * (p1.y - p0.y)
    const by = 3 * (p2.y - p1.y) - cy
    const ay = p3.y - p0.y - cy - by

    const x = ax * (t ** 3) + bx * (t ** 2) + cx * t + p0.x
    const y = ay * (t ** 3) + by * (t ** 2) + cy * t + p0.y

    return { x, y }
  }

  function frame(now) {
    const elapsed = now - startTime
    const t = Math.min(1, elapsed / duration)

    ctx.clearRect(0, 0, canvas.width, canvas.height)

    // Si la estrella viaja, dibujar su núcleo de luz y generar polvo de hadas
    if (t < 1) {
      const pos = getArcPos(t)

      // Núcleo ultrabrillante
      ctx.save()
      ctx.shadowBlur = 30
      ctx.shadowColor = '#ffffff'
      ctx.fillStyle = '#ffffff'
      ctx.beginPath()
      ctx.arc(pos.x, pos.y, 5, 0, Math.PI * 2)
      ctx.fill()

      // Resplandor dorado / cian
      ctx.shadowBlur = 45
      ctx.shadowColor = '#ffdf7a'
      ctx.fillStyle = 'rgba(255, 240, 160, 0.7)'
      ctx.beginPath()
      ctx.arc(pos.x, pos.y, 10, 0, Math.PI * 2)
      ctx.fill()

      // Cruz de destello
      ctx.strokeStyle = 'rgba(255, 255, 255, 0.8)'
      ctx.lineWidth = 1.5
      ctx.beginPath()
      ctx.moveTo(pos.x - 16, pos.y)
      ctx.lineTo(pos.x + 16, pos.y)
      ctx.moveTo(pos.x, pos.y - 16)
      ctx.lineTo(pos.x, pos.y + 16)
      ctx.stroke()
      ctx.restore()

      // Emisión de polvo de hadas (Tinkerbell stardust)
      for (let i = 0; i < 6; i++) {
        starParticles.push({
          x: pos.x + (Math.random() - 0.5) * 8,
          y: pos.y + (Math.random() - 0.5) * 8,
          vx: (Math.random() - 0.5) * 1.6,
          vy: Math.random() * 1.4 + 0.3,
          size: Math.random() * 3.2 + 1,
          alpha: 1,
          decay: 0.007 + Math.random() * 0.012,
          color: Math.random() > 0.4 ? '#fffde8' : (Math.random() > 0.5 ? '#7de5ff' : '#ffcf4d'),
        })
      }
    }

    // Actualizar y dibujar partículas
    for (let i = starParticles.length - 1; i >= 0; i--) {
      const p = starParticles[i]
      p.x += p.vx
      p.y += p.vy
      p.alpha -= p.decay
      p.size = Math.max(0, p.size - p.decay * 1.2)

      if (p.alpha <= 0) {
        starParticles.splice(i, 1)
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

    if (t < 1 || starParticles.length > 0) {
      animId = requestAnimationFrame(frame)
    } else {
      // Repetir el arco cada 12 segundos para mantener el show activo
      starCycleTimer = setTimeout(() => {
        if (etapa.value === 'final') lanzarEstrellaDisney()
      }, 9000)
    }
  }

  animId = requestAnimationFrame(frame)
}

function burstDisney(x, y) {
  confetti({
    particleCount: 32,
    spread: 85,
    startVelocity: 34,
    origin: { x, y },
    colors,
    shapes: [heartShape],
    scalar: 1.4,
    ticks: 200,
    gravity: 0.8,
    disableForReducedMotion: true,
  })
}

function iniciarFuegosDisney() {
  fireworksTimer = setInterval(() => {
    if (Math.random() > 0.35) {
      burstDisney(
        Math.random() > 0.5 ? 0.16 + Math.random() * 0.2 : 0.64 + Math.random() * 0.2,
        0.18 + Math.random() * 0.25
      )
    }
  }, 1600)
}

function handlePointer(e) {
  if (etapa.value !== 'final') return
  const x = e.clientX || (e.touches && e.touches[0]?.clientX)
  const y = e.clientY || (e.touches && e.touches[0]?.clientY)
  if (x != null && y != null && Math.random() > 0.3) {
    for (let i = 0; i < 2; i++) {
      starParticles.push({
        x: x + (Math.random() - 0.5) * 10,
        y: y + (Math.random() - 0.5) * 10,
        vx: (Math.random() - 0.5) * 1.5,
        vy: (Math.random() - 0.5) * 1.5 - 0.5,
        size: Math.random() * 2.5 + 1.2,
        alpha: 0.9,
        decay: 0.02,
        color: Math.random() > 0.5 ? '#fff4b8' : '#7fe4ff',
      })
    }
  }
}

onBeforeUnmount(() => {
  if (fireworksTimer) clearInterval(fireworksTimer)
  if (starCycleTimer) clearTimeout(starCycleTimer)
  if (animId) cancelAnimationFrame(animId)
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

  <!-- ════════ Etapa 3: FINAL ÉPICO DISNEY (Intro de película) ════════ -->
  <section
    v-else
    class="scene epic-disney-scene"
    @pointermove="handlePointer"
    @touchmove.passive="handlePointer"
  >
    <!-- Fondo fotográfico hiperrealista del Castillo de Disney con movimiento de cámara (estilo video) -->
    <div class="disney-video-bg">
      <img
        :src="disneyCastleImg"
        alt="Castillo de Disney con fuegos artificiales y arco de estrella fugaz"
        class="disney-castle-photo"
      />
    </div>

    <!-- Bruma y resplandor mágico sobre el lago -->
    <div class="disney-aura" aria-hidden="true"></div>

    <!-- Canvas para la estrella fugaz en vivo y polvo de hadas de Campanita -->
    <canvas ref="arcCanvasRef" class="disney-fx-canvas" aria-hidden="true"></canvas>

    <!-- ══ TEXTO FINAL CINEMÁTICO DISNEY ══ -->
    <div class="disney-text-overlay">
      <p class="disney-prelude">Y así, después de tanto amor…</p>
      <h1 class="disney-main-title">Y vivieron felices<br />para siempre</h1>
      <p class="disney-closure">El fin… de esta carta. Nunca de nosotros. <span class="infinity">♾️</span></p>

      <div class="replay-zone">
        <button class="btn-replay" @click="reiniciar">
          <span>↺</span> Volver al inicio
        </button>
      </div>
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

/* ──────────────────────────────────────────────────────────
   ETAPA 3: FINAL ÉPICO DISNEY (Estilo video / intro de cine)
────────────────────────────────────────────────────────── */
.epic-disney-scene {
  position: fixed;
  inset: 0;
  width: 100vw;
  height: 100svh;
  overflow: hidden;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #030514;
}

/* Fondo cinemático del castillo con movimiento de video (Ken Burns) */
.disney-video-bg {
  position: absolute;
  inset: -4%;
  width: 108%;
  height: 108%;
  overflow: hidden;
  will-change: transform;
}

.disney-castle-photo {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: 50% 45%;
  display: block;
  animation: disneyVideoCam 28s ease-in-out infinite alternate;
  transform-origin: 50% 40%;
}

@keyframes disneyVideoCam {
  0% {
    transform: scale(1) translate3d(0, 0, 0);
  }
  50% {
    transform: scale(1.06) translate3d(-0.6%, -0.4%, 0);
  }
  100% {
    transform: scale(1.03) translate3d(0.5%, 0.3%, 0);
  }
}

/* Bruma sutil y halo de luz en la base */
.disney-aura {
  position: absolute;
  inset: 0;
  background: radial-gradient(circle at 50% 50%, rgba(255, 255, 255, 0.05) 0%, transparent 70%),
              linear-gradient(to top, rgba(3, 5, 20, 0.4) 0%, transparent 40%);
  pointer-events: none;
  z-index: 2;
}

/* Canvas del polvo de hadas y estrella fugaz */
.disney-fx-canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 10;
}

/* ── TEXTO FINAL DISNEY EN EL CIELO ── */
.disney-text-overlay {
  position: relative;
  z-index: 20;
  text-align: center;
  color: #ffffff;
  padding: clamp(20px, 4vw, 32px) clamp(22px, 5vw, 40px);
  max-width: min(760px, 92vw);
  border-radius: 24px;
  background: radial-gradient(ellipse at center, rgba(6, 10, 26, 0.82) 0%, rgba(6, 10, 26, 0.6) 65%, rgba(6, 10, 26, 0.25) 100%);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  border: 1px solid rgba(255, 235, 170, 0.3);
  box-shadow: 0 16px 50px rgba(0, 0, 0, 0.7);
  pointer-events: none;
  animation: disneyTextReveal 2.4s cubic-bezier(0.16, 1, 0.3, 1) 0.5s both;
  margin: 0 auto;
}

@keyframes disneyTextReveal {
  0% {
    opacity: 0;
    transform: translateY(22px) scale(0.96);
    filter: blur(6px);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
    filter: blur(0);
  }
}

.disney-prelude {
  margin: 0 0 12px;
  font-family: 'Cinzel', 'Playfair Display', Georgia, serif;
  font-weight: 700;
  font-size: clamp(1rem, 2.8vw, 1.45rem);
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #ffe699;
  text-shadow:
    0 2px 8px rgba(0, 0, 0, 0.95),
    0 0 18px rgba(255, 220, 100, 0.6);
}

.disney-main-title {
  margin: 0;
  font-family: 'Cinzel', 'Playfair Display', Georgia, serif;
  font-weight: 800;
  font-size: clamp(1.9rem, 5.8vw, 3.6rem);
  line-height: 1.22;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  background: linear-gradient(180deg, #ffffff 0%, #fff6d8 45%, #ffd566 100%);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  filter: drop-shadow(0 4px 14px rgba(0, 0, 0, 0.95)) drop-shadow(0 0 28px rgba(255, 220, 100, 0.65));
}

.disney-closure {
  margin: 18px 0 0;
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-style: italic;
  font-size: clamp(1.05rem, 2.7vw, 1.35rem);
  letter-spacing: 0.05em;
  color: #fff9eb;
  text-shadow:
    0 2px 8px rgba(0, 0, 0, 0.95),
    0 0 16px rgba(255, 230, 150, 0.5);
}

.infinity {
  display: inline-block;
  font-style: normal;
  color: #ffe685;
  filter: drop-shadow(0 0 8px #ffe685);
}

.replay-zone {
  margin-top: 22px;
  pointer-events: auto;
}

.btn-replay {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 26px;
  font-family: 'Cinzel', serif;
  font-size: 0.92rem;
  font-weight: 600;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #fff9e6;
  background: rgba(12, 18, 40, 0.7);
  border: 1px solid rgba(255, 230, 150, 0.5);
  border-radius: 999px;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.5);
  cursor: pointer;
  transition: transform 0.25s ease, background 0.25s ease, border-color 0.25s ease;
}

.btn-replay:hover {
  transform: scale(1.05);
  background: rgba(25, 35, 75, 0.85);
  border-color: rgba(255, 240, 180, 0.9);
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

  .disney-text-overlay {
    padding: 18px 18px;
    max-width: 94vw;
  }

  .disney-main-title {
    font-size: clamp(1.65rem, 6.5vw, 2.4rem);
  }

  .disney-prelude {
    font-size: 0.95rem;
    letter-spacing: 0.1em;
  }

  .disney-closure {
    font-size: 0.98rem;
  }
}

@media (max-height: 640px) {
  .disney-text-overlay {
    padding: 14px 18px;
  }
}
</style>
