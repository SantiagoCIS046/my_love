<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import confetti from 'canvas-confetti'
import { cartaFinal, firmaCarta } from '../data/content'
import { config } from '../data/config'

const etapa = ref('entrada') // entrada -> carta -> final
const arcCanvasRef = ref(null)
const starActive = ref(false)

const heartShape = confetti.shapeFromPath({
  path: 'M167 72c19,-38 37,-56 75,-56 42,0 76,33 76,75 0,76 -76,151 -151,227 -76,-76 -151,-151 -151,-227 0,-42 33,-75 75,-75 38,0 57,18 76,56z',
})

const colors = ['#fde047', '#f472b6', '#67e8f9', '#ffffff', '#e879f9', '#fbbf24']
let fireworksTimer = null
let starAnimTimer = null

// Estrellas titilantes en el cielo nocturno
const stars = Array.from({ length: 85 }, (_, s) => {
  const r = ((s * 9301 + 49297) % 233280) / 233280
  const r2 = ((s * 4801 + 1723) % 10000) / 10000
  return {
    left: (r * 100).toFixed(2) + '%',
    top: (r2 * 68).toFixed(2) + '%',
    size: (1 + r2 * 2.2).toFixed(1) + 'px',
    animationDelay: (r2 * 4).toFixed(2) + 's',
    animationDuration: (2.5 + r * 3.5).toFixed(2) + 's',
  }
})

function abrirCarta() {
  etapa.value = 'carta'
}

function granFinal() {
  etapa.value = 'final'
  // Disparar la estrella fugaz icónica de Disney tras 0.6s
  setTimeout(() => {
    lanzarEstrellaDisney()
    iniciarFuegosDisney()
  }, 600)
}

// ── ESTRELLA FUGAZ DISNEY & POLVO DE HADAS (Stardust) ──
let animId = null
let starParticles = []

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

  // Duración del arco: 3.2s
  const startTime = performance.now()
  const duration = 3200

  // Parábola del arco Disney sobre el castillo
  // t va de 0 a 1
  function getArcPos(t) {
    const w = canvas.width
    const h = canvas.height
    // Inicio: 8% ancho, 75% alto | Vértice: 50% ancho, 14% alto | Fin: 92% ancho, 75% alto
    const p0 = { x: w * 0.08, y: h * 0.72 }
    const p1 = { x: w * 0.32, y: h * 0.12 }
    const p2 = { x: w * 0.68, y: h * 0.12 }
    const p3 = { x: w * 0.92, y: h * 0.72 }

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

    // Si la estrella sigue en vuelo, generar chispas en su posición actual
    if (t < 1) {
      const pos = getArcPos(t)

      // Cabeza brillante de la estrella
      ctx.save()
      ctx.shadowBlur = 25
      ctx.shadowColor = '#fffdf0'
      ctx.fillStyle = '#ffffff'
      ctx.beginPath()
      ctx.arc(pos.x, pos.y, 4.5, 0, Math.PI * 2)
      ctx.fill()

      // Destello exterior dorado
      ctx.shadowBlur = 40
      ctx.shadowColor = '#ffe26a'
      ctx.fillStyle = 'rgba(255, 240, 150, 0.6)'
      ctx.beginPath()
      ctx.arc(pos.x, pos.y, 8, 0, Math.PI * 2)
      ctx.fill()
      ctx.restore()

      // Spawn de partículas de estela (polvo de hadas)
      for (let i = 0; i < 4; i++) {
        starParticles.push({
          x: pos.x + (Math.random() - 0.5) * 6,
          y: pos.y + (Math.random() - 0.5) * 6,
          vx: (Math.random() - 0.5) * 1.5,
          vy: Math.random() * 1.2 + 0.4,
          size: Math.random() * 2.8 + 1,
          alpha: 1,
          decay: 0.008 + Math.random() * 0.012,
          color: Math.random() > 0.35 ? '#fffae0' : (Math.random() > 0.5 ? '#93e8ff' : '#ffd060'),
        })
      }
    }

    // Dibujar y actualizar partículas
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
      ctx.shadowBlur = 8
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
      // Repetir el arco mágico cada 14 segundos para mantener la magia viva
      starAnimTimer = setTimeout(() => {
        if (etapa.value === 'final') lanzarEstrellaDisney()
      }, 10000)
    }
  }

  animId = requestAnimationFrame(frame)
}

function burstDisney(x, y) {
  confetti({
    particleCount: 30,
    spread: 80,
    startVelocity: 32,
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
  // Destellos mágicos suaves en el cielo a los lados del castillo
  fireworksTimer = setInterval(() => {
    if (Math.random() > 0.3) {
      burstDisney(
        Math.random() > 0.5 ? 0.15 + Math.random() * 0.22 : 0.63 + Math.random() * 0.22,
        0.18 + Math.random() * 0.28
      )
    }
  }, 1400)
}

onBeforeUnmount(() => {
  if (fireworksTimer) clearInterval(fireworksTimer)
  if (starAnimTimer) clearTimeout(starAnimTimer)
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

  <!-- ════════ Etapa 3: final estilo Disney ════════ -->
  <section v-else class="scene disney-stage">
    <!-- Fondo de cielo estrellado con gradiente Disney crepuscular -->
    <div class="stars" aria-hidden="true">
      <span
        v-for="(s, i) in stars"
        :key="i"
        class="star"
        :style="{
          left: s.left,
          top: s.top,
          width: s.size,
          height: s.size,
          animationDelay: s.animationDelay,
          animationDuration: s.animationDuration,
        }"
      ></span>
    </div>

    <!-- Luna mágica -->
    <div class="moon" aria-hidden="true"></div>

    <!-- Canvas para la estela de polvo de hadas de la estrella fugaz -->
    <canvas ref="arcCanvasRef" class="stardust-canvas" aria-hidden="true"></canvas>

    <!-- ══ EL ICÓNICO CASTILLO DE DISNEY CON ILUMINACIÓN MÁGICA ══ -->
    <div class="disney-castle-wrap" aria-hidden="true">
      <!-- Aura de luz detrás del castillo -->
      <div class="castle-backlight"></div>

      <!-- Silueta vectorial detallada del castillo de Cenicienta / Disney -->
      <svg
        class="disney-castle-svg"
        viewBox="0 0 1000 680"
        preserveAspectRatio="xMidYMax meet"
      >
        <defs>
          <linearGradient id="castleGrad" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0%"   stop-color="#24143e" />
            <stop offset="40%"  stop-color="#180e2f" />
            <stop offset="100%" stop-color="#0b0617" />
          </linearGradient>

          <linearGradient id="castleHighlight" x1="0" y1="0" x2="1" y2="0">
            <stop offset="0%"   stop-color="#3c2062" />
            <stop offset="50%"  stop-color="#552f82" />
            <stop offset="100%" stop-color="#24143e" />
          </linearGradient>

          <radialGradient id="windowLight" cx="50%" cy="50%" r="50%">
            <stop offset="0%"   stop-color="#fffce8" />
            <stop offset="60%"  stop-color="#ffd56a" />
            <stop offset="100%" stop-color="#e09520" />
          </radialGradient>

          <radialGradient id="portalLight" cx="50%" cy="75%" r="65%">
            <stop offset="0%"   stop-color="#fff8d5" stop-opacity="0.9" />
            <stop offset="45%"  stop-color="#f5c250" stop-opacity="0.6" />
            <stop offset="100%" stop-color="#3c1550" stop-opacity="0" />
          </radialGradient>

          <filter id="windowGlow" x="-50%" y="-50%" width="200%" height="200%">
            <feGaussianBlur in="SourceGraphic" stdDeviation="4" result="blur" />
            <feMerge>
              <feMergeNode in="blur" />
              <feMergeNode in="SourceGraphic" />
            </feMerge>
          </filter>
        </defs>

        <!-- ── Resplandor del portal central ── -->
        <path d="M 450,680 L 450,565 C 450,530 550,530 550,565 L 550,680 Z" fill="url(#portalLight)" />

        <!-- ── Silueta completa del Castillo de Disney ── -->
        <g fill="url(#castleGrad)">
          <!-- Base muralla exterior y almenas -->
          <rect x="180" y="610" width="640" height="70" />
          <!-- Almenas inferiores -->
          <path d="
            M 180,610 L 180,590 L 210,590 L 210,610 L 230,610 L 230,590 L 260,590 L 260,610
            L 280,610 L 280,590 L 310,590 L 310,610 L 330,610 L 330,590 L 360,590 L 360,610
            L 640,610 L 640,590 L 670,590 L 670,610 L 690,610 L 690,590 L 720,590 L 720,610
            L 740,610 L 740,590 L 770,590 L 770,610 L 790,610 L 790,590 L 820,590 L 820,610 Z
          " />

          <!-- Muralla media izquierda y derecha -->
          <rect x="250" y="520" width="130" height="90" />
          <rect x="620" y="520" width="130" height="90" />

          <!-- Torre lateral lejana izquierda -->
          <rect x="190" y="470" width="45" height="140" />
          <polygon points="180,470 212,370 245,470" fill="url(#castleHighlight)" />
          <polygon points="212,370 212,350 220,358" fill="#ffd56a" />

          <!-- Torre lateral lejana derecha -->
          <rect x="765" y="470" width="45" height="140" />
          <polygon points="755,470 788,370 820,470" fill="url(#castleHighlight)" />
          <polygon points="788,370 788,350 796,358" fill="#ffd56a" />

          <!-- Torre media izquierda (grande) -->
          <rect x="280" y="410" width="65" height="180" />
          <polygon points="270,410 312,280 355,410" fill="url(#castleHighlight)" />
          <polygon points="312,280 312,258 322,266" fill="#ffd56a" />

          <!-- Torre media derecha (grande) -->
          <rect x="655" y="410" width="65" height="180" />
          <polygon points="645,410 688,280 730,410" fill="url(#castleHighlight)" />
          <polygon points="688,280 688,258 698,266" fill="#ffd56a" />

          <!-- Cuerpo central inferior -->
          <rect x="360" y="460" width="280" height="150" />

          <!-- Arbotantes laterales (flying buttresses icónicos) -->
          <path d="M 360,520 Q 330,470 340,430 L 350,430 Q 345,480 360,500 Z" fill="url(#castleHighlight)" />
          <path d="M 640,520 Q 670,470 660,430 L 650,430 Q 655,480 640,500 Z" fill="url(#castleHighlight)" />

          <!-- Segundo nivel central -->
          <rect x="400" y="380" width="200" height="120" />
          <!-- Almenas del segundo nivel -->
          <path d="
            M 390,380 L 390,365 L 415,365 L 415,380 L 430,380 L 430,365 L 455,365 L 455,380
            L 545,380 L 545,365 L 570,365 L 570,380 L 585,380 L 585,365 L 610,365 L 610,380 Z
          " />

          <!-- Torre central media izquierda y derecha -->
          <rect x="415" y="290" width="40" height="90" />
          <polygon points="408,290 435,195 462,290" fill="url(#castleHighlight)" />

          <rect x="545" y="290" width="40" height="90" />
          <polygon points="538,290 565,195 592,290" fill="url(#castleHighlight)" />

          <!-- ══ GRAN TORRE CENTRAL PRINCIPAL DE CENICIENTA ══ -->
          <rect x="465" y="240" width="70" height="160" />
          <!-- Galería circular de la aguja -->
          <rect x="455" y="235" width="90" height="14" rx="3" />
          <!-- Aguja cónica monumental -->
          <polygon points="450,235 500,45 550,235" fill="url(#castleHighlight)" />
          <!-- Banderín ondeando en la cima más alta -->
          <polygon points="500,45 500,20 522,30 500,38" fill="#ffe26a" />

          <!-- Torrecillas adosadas a la aguja central -->
          <rect x="472" y="200" width="12" height="40" />
          <polygon points="470,200 478,160 486,200" fill="#4d2876" />
          <rect x="516" y="200" width="12" height="40" />
          <polygon points="514,200 522,160 530,200" fill="#4d2876" />

          <!-- Portal principal en arco -->
          <path d="M 465,680 L 465,580 C 465,550 535,550 535,580 L 535,680 Z" fill="#07030e" />
          <path d="M 470,680 L 470,582 C 470,556 530,556 530,582 L 530,680 Z" fill="#ffca58" opacity="0.85" filter="url(#windowGlow)" />
        </g>

        <!-- ── Ventanas cálidas iluminadas con velas ── -->
        <g fill="url(#windowLight)" filter="url(#windowGlow)">
          <!-- Ventana aguja principal -->
          <path d="M 495,190 C 495,182 505,182 505,190 L 505,210 L 495,210 Z" />
          <!-- Ventana gran torre central -->
          <path d="M 490,290 C 490,278 510,278 510,290 L 510,325 L 490,325 Z" />
          <!-- Ventanas torres intermedias -->
          <path d="M 428,320 C 428,312 442,312 442,320 L 442,345 L 428,345 Z" />
          <path d="M 558,320 C 558,312 572,312 572,320 L 572,345 L 558,345 Z" />
          <!-- Ventanas torres laterales grandes -->
          <path d="M 305,445 C 305,435 320,435 320,445 L 320,475 L 305,475 Z" />
          <path d="M 680,445 C 680,435 695,435 695,445 L 695,475 L 680,475 Z" />
          <!-- Ventanas torres lejanas -->
          <path d="M 205,500 C 205,492 218,492 218,500 L 218,525 L 205,525 Z" />
          <path d="M 780,500 C 780,492 793,492 793,500 L 793,525 L 780,525 Z" />
          <!-- Ventanas cuerpo medio -->
          <circle cx="475" cy="425" r="7" />
          <circle cx="525" cy="425" r="7" />
        </g>
      </svg>
    </div>

    <!-- ══ TEXTO FINAL ROMÁNTICO ESTILO DISNEY ══ -->
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

/* ──────────────────────────────────────────────────────────
   ETAPA 3: FINAL DISNEY AUTÉNTICO
────────────────────────────────────────────────────────── */
.disney-stage {
  display: flex;
  align-items: center;
  justify-content: center;
  background:
    radial-gradient(ellipse at 50% 90%, rgba(240, 150, 170, 0.4) 0%, rgba(95, 45, 125, 0.3) 45%, transparent 70%),
    linear-gradient(180deg, #070924 0%, #12143d 35%, #25164d 68%, #421844 100%);
  overflow: hidden;
  position: relative;
}

/* Canvas del Polvo de Hadas y Estrella Fugaz */
.stardust-canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 15;
}

/* Luna llena Disney */
.moon {
  position: absolute;
  top: 8%;
  right: 12%;
  width: clamp(65px, 9vw, 95px);
  aspect-ratio: 1;
  border-radius: 50%;
  background: radial-gradient(circle at 38% 35%, #ffffff 0%, #fff6d6 55%, #f0d588 100%);
  box-shadow:
    0 0 35px rgba(255, 245, 200, 0.6),
    0 0 90px rgba(255, 240, 180, 0.3);
  animation: moonFade 2s ease 0.3s both;
  z-index: 2;
}

@keyframes moonFade {
  from { opacity: 0; transform: translateY(-15px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Estrellas titilantes */
.stars {
  position: absolute;
  inset: 0;
  z-index: 1;
  pointer-events: none;
}

.star {
  position: absolute;
  border-radius: 50%;
  background: #ffffff;
  animation: starBlink linear infinite;
}

@keyframes starBlink {
  0%, 100% { opacity: 0.2; transform: scale(0.9); }
  50%      { opacity: 1;   transform: scale(1.4); }
}

/* ── CASTILLO DE DISNEY ── */
.disney-castle-wrap {
  position: absolute;
  bottom: -4px;
  left: 50%;
  transform: translateX(-50%);
  width: min(840px, 98vw);
  height: clamp(280px, 48vh, 520px);
  display: flex;
  align-items: flex-end;
  justify-content: center;
  z-index: 4;
  pointer-events: none;
  animation: castleRise 2.4s cubic-bezier(0.16, 1, 0.3, 1) both;
}

@keyframes castleRise {
  from { opacity: 0; transform: translateX(-50%) translateY(40px); }
  to   { opacity: 1; transform: translateX(-50%) translateY(0); }
}

.castle-backlight {
  position: absolute;
  bottom: 12%;
  left: 50%;
  transform: translateX(-50%);
  width: 75%;
  height: 65%;
  background: radial-gradient(ellipse at 50% 60%, rgba(245, 200, 110, 0.35) 0%, rgba(200, 110, 170, 0.2) 50%, transparent 80%);
  filter: blur(28px);
  pointer-events: none;
}

.disney-castle-svg {
  width: 100%;
  height: 100%;
  display: block;
  filter: drop-shadow(0 -8px 25px rgba(25, 10, 45, 0.6));
}

/* ── TEXTO FINAL DISNEY ── */
.finale-text {
  position: relative;
  z-index: 10;
  text-align: center;
  color: #fff9f0;
  margin-top: -6vh;
  animation: textReveal 2.5s cubic-bezier(0.22, 1, 0.36, 1) 0.5s both;
  pointer-events: none;
}

@keyframes textReveal {
  from { opacity: 0; transform: translateY(22px) scale(0.96); }
  to   { opacity: 1; transform: translateY(0) scale(1); }
}

.finale-script {
  margin: 0 0 10px;
  font-family: var(--font-script);
  font-size: clamp(1.4rem, 3.8vw, 2.1rem);
  color: #fce7f3;
  text-shadow: 0 2px 14px rgba(0, 0, 0, 0.7);
}

.finale-title {
  margin: 0;
  font-family: var(--font-title);
  font-weight: 400;
  font-size: clamp(2.8rem, 7.8vw, 5.4rem);
  line-height: 1.15;
  color: #ffffff;
  text-shadow:
    0 0 25px rgba(255, 235, 170, 0.9),
    0 0 60px rgba(245, 190, 80, 0.5),
    0 4px 25px rgba(0, 0, 0, 0.75);
}

.finale-sub {
  margin: 20px 0 0;
  font-style: italic;
  font-size: clamp(1rem, 2.6vw, 1.25rem);
  color: rgba(255, 248, 235, 0.9);
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.85);
  letter-spacing: 0.04em;
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
    top: 6%;
    right: 8%;
  }

  .disney-castle-wrap {
    width: 100vw;
    height: 40vh;
  }

  .finale-text {
    margin-top: -14vh;
    padding: 0 16px;
  }

  .finale-title {
    font-size: clamp(2.3rem, 9vw, 3.2rem);
  }
}

@media (max-height: 640px) {
  .disney-castle-wrap {
    height: 32vh;
  }
  .finale-text {
    margin-top: -10vh;
  }
}
</style>
