<script setup>
import { ref } from 'vue'

const emit = defineEmits(['start', 'next'])
const zooming = ref(false)

// Escalera en perspectiva: sube de abajo-derecha hacia la luz del atardecer
const steps = Array.from({ length: 16 }, (_, i) => {
  const t = i / 15
  return {
    x: 268 - t * 158,
    y: 640 - t * 300,
    w: 64 * (1 - t * 0.42),
    h: 11 * (1 - t * 0.3),
    o: (1 - t * 0.55).toFixed(2),
  }
})

function agarrar() {
  if (zooming.value) return
  zooming.value = true
  emit('start') // la app se prepara debajo mientras hacemos el zoom
  setTimeout(() => emit('next'), 1500)
}
</script>

<template>
  <section class="intro" :class="{ zooming }" @click="agarrar">
    <!-- ══ Paisaje original: atardecer con escaleras subiendo ══ -->
    <svg class="landscape" viewBox="0 0 400 700" preserveAspectRatio="xMidYMax slice" aria-hidden="true">
      <defs>
        <linearGradient id="sky" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" stop-color="#3b1757" />
          <stop offset="30%" stop-color="#7a2560" />
          <stop offset="55%" stop-color="#c14a63" />
          <stop offset="75%" stop-color="#ef7d54" />
          <stop offset="100%" stop-color="#ffb35c" />
        </linearGradient>
        <radialGradient id="sunGlow" cx="50%" cy="50%" r="50%">
          <stop offset="0%" stop-color="#fff3c4" stop-opacity="0.95" />
          <stop offset="35%" stop-color="#ffcf87" stop-opacity="0.55" />
          <stop offset="100%" stop-color="#ffcf87" stop-opacity="0" />
        </radialGradient>
        <linearGradient id="hillFar" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" stop-color="#6e2a4e" />
          <stop offset="100%" stop-color="#521d3e" />
        </linearGradient>
        <linearGradient id="hillNear" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" stop-color="#46163a" />
          <stop offset="100%" stop-color="#2d0e28" />
        </linearGradient>
        <linearGradient id="stepTop" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" stop-color="#ffc987" />
          <stop offset="100%" stop-color="#e08a54" />
        </linearGradient>
      </defs>

      <!-- Cielo -->
      <rect width="400" height="700" fill="url(#sky)" />

      <!-- Sol poniente sobre el horizonte -->
      <circle cx="150" cy="430" r="170" fill="url(#sunGlow)" />
      <circle cx="150" cy="430" r="46" fill="#fff0bd" opacity="0.95" />

      <!-- Nubes suaves teñidas de dorado -->
      <ellipse cx="90" cy="250" rx="90" ry="12" fill="#f2a06b" opacity="0.35" />
      <ellipse cx="300" cy="200" rx="110" ry="10" fill="#e98a72" opacity="0.3" />
      <ellipse cx="210" cy="310" rx="130" ry="9" fill="#ffbe7d" opacity="0.4" />
      <ellipse cx="60" cy="150" rx="70" ry="8" fill="#c06a86" opacity="0.3" />

      <!-- Colinas lejanas -->
      <path d="M0 460 Q 70 400 150 445 T 400 430 V 700 H 0 Z" fill="url(#hillFar)" opacity="0.85" />

      <!-- Colina principal con la escalera -->
      <path d="M0 520 Q 120 470 230 505 T 400 500 V 700 H 0 Z" fill="url(#hillNear)" />

      <!-- Escaleras de piedra subiendo hacia la luz -->
      <g v-for="(s, i) in steps" :key="i">
        <rect :x="s.x" :y="s.y" :width="s.w" :height="s.h * 1.6" rx="2" fill="#2a0c24" :opacity="s.o" />
        <rect :x="s.x" :y="s.y" :width="s.w" :height="s.h" rx="2" fill="url(#stepTop)" :opacity="s.o" />
      </g>

      <!-- Lado de la escalera: baranda de piedra simple -->
      <path
        d="M280 640 L 118 348"
        stroke="#20081e"
        stroke-width="7"
        stroke-linecap="round"
        opacity="0.9"
        fill="none"
      />

      <!-- Vegetación en silueta -->
      <path d="M0 700 Q 40 640 20 600 Q 60 650 70 700 Z" fill="#230a22" />
      <path d="M400 700 Q 350 630 372 585 Q 330 655 322 700 Z" fill="#230a22" />
      <path d="M310 700 Q 335 660 328 630 Q 355 672 358 700 Z" fill="#1c071c" />
    </svg>

    <!-- ══ Contenido: las manos + la frase ══ -->
    <div class="veil" aria-hidden="true"></div>

    <div class="content fade-up">
      <button class="hands-btn" aria-label="Agárrame de la mano" @click.stop="agarrar">
        <span class="hands-glow" aria-hidden="true"></span>
        <img
          class="hands"
          src="/photos/manos.jpg"
          alt="Dos manos morenas tomadas de la mano al atardecer"
        />
        <span class="tap-hint" aria-hidden="true">tócalas&nbsp;✦</span>
      </button>

      <h1 class="invite script-title">Agárrame de la mano,</h1>
      <p class="invite-sub">aquí empieza nuestra historia</p>
    </div>
  </section>
</template>

<style scoped>
.intro {
  position: fixed;
  inset: 0;
  z-index: 50;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  cursor: pointer;
  background: #3b1757;
  /* Zoom cinematográfico hacia el atardecer */
  transition:
    transform 1.5s cubic-bezier(0.4, 0, 0.2, 1),
    filter 1.4s ease-in,
    opacity 1.3s ease-in;
  transform-origin: 50% 42%;
}

.intro.zooming {
  transform: scale(2.7);
  filter: blur(16px) brightness(1.25);
  opacity: 0;
  pointer-events: none;
}

.landscape {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
}

/* Velo suave para legibilidad del texto */
.veil {
  position: absolute;
  inset: 0;
  background:
    radial-gradient(ellipse at 50% 40%, rgba(59, 23, 87, 0.1) 0%, rgba(45, 14, 40, 0.42) 100%),
    linear-gradient(180deg, rgba(59, 23, 87, 0.25) 0%, transparent 30%, transparent 62%, rgba(30, 8, 30, 0.6) 100%);
}

.content {
  position: relative;
  z-index: 2;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  padding: 24px;
}

/* ---------- Las manos ---------- */
.hands-btn {
  position: relative;
  background: none;
  border: none;
  padding: 0;
  margin-bottom: 30px;
  animation: floatHands 5s ease-in-out infinite;
}

.hands {
  display: block;
  width: clamp(220px, 62vw, 320px);
  aspect-ratio: 1;
  object-fit: cover;
  border-radius: 50%;
  border: 4px solid rgba(255, 243, 214, 0.85);
  box-shadow:
    0 0 0 8px rgba(255, 205, 135, 0.12),
    0 0 60px rgba(255, 190, 120, 0.45),
    0 18px 50px rgba(30, 8, 30, 0.55);
  transition: transform 0.35s cubic-bezier(0.22, 1, 0.36, 1), box-shadow 0.35s ease;
}

.hands-btn:active .hands {
  transform: scale(0.96);
}

.hands-glow {
  position: absolute;
  inset: -22px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(255, 205, 135, 0.35) 0%, transparent 70%);
  animation: pulseGlow 3.2s ease-in-out infinite;
  pointer-events: none;
}

.tap-hint {
  position: absolute;
  bottom: -6px;
  right: -4px;
  font-family: var(--font-script);
  font-size: 1.15rem;
  color: #fff3d6;
  background: rgba(74, 15, 30, 0.72);
  padding: 5px 14px;
  border-radius: 999px;
  border: 1px solid rgba(255, 243, 214, 0.4);
  box-shadow: 0 4px 14px rgba(30, 8, 30, 0.45);
  animation: pulseGlow 2.6s ease-in-out infinite;
}

/* ---------- Frase ---------- */
.invite {
  margin: 0;
  font-size: clamp(2.2rem, 8vw, 3.2rem);
  line-height: 1.15;
  color: #fff3d6;
  text-shadow:
    0 0 22px rgba(255, 190, 120, 0.5),
    0 3px 18px rgba(30, 8, 30, 0.65);
}

.invite-sub {
  margin: 12px 0 0;
  font-family: var(--font-script);
  font-size: clamp(1.3rem, 4.5vw, 1.7rem);
  color: #ffc9a0;
  text-shadow: 0 2px 14px rgba(30, 8, 30, 0.6);
}

/* ---------- Animaciones ---------- */
@keyframes floatHands {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

@keyframes pulseGlow {
  0%, 100% { opacity: 0.55; }
  50% { opacity: 1; }
}

@media (max-width: 640px) {
  .content {
    padding: 18px 16px;
  }

  .hands-btn {
    margin-bottom: 22px;
  }

  .invite-sub {
    padding: 0 10px;
  }
}

@media (max-height: 620px) {
  .hands {
    width: clamp(180px, 46vw, 240px);
  }

  .hands-btn {
    margin-bottom: 14px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .hands-btn,
  .hands-glow,
  .tap-hint {
    animation: none;
  }

  .intro {
    transition: opacity 0.4s ease;
  }
}
</style>
