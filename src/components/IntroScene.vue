<script setup>
import { ref } from 'vue'

const emit = defineEmits(['start', 'next'])
const volando = ref(false)

const lunaEl = ref(null)
const luciernagas = ref([])

// Estrellas deterministas (parpadeo natural)
const stars = Array.from({ length: 80 }, (_, i) => {
  const r = ((i * 9301 + 49297) % 233280) / 233280
  const r2 = ((i * 4801 + 1723) % 10000) / 10000
  return {
    left: (r * 100).toFixed(2) + '%',
    top: (r2 * 62).toFixed(2) + '%',
    size: (1 + r2 * 2).toFixed(1) + 'px',
    animationDelay: (r2 * 5).toFixed(2) + 's',
    animationDuration: (2.5 + r * 4).toFixed(2) + 's',
  }
})

function setLuci(el, i) {
  luciernagas.value[i] = el
}

function tocar() {
  if (volando.value) return
  volando.value = true
  emit('start') // la historia se prepara debajo del vuelo

  let ultimo = 0
  luciernagas.value.slice(0, 2).forEach((el, i) => {
    ultimo = Math.max(ultimo, volarHaciaLaLuna(el, i * 450))
  })

  // La luna las recibe con un latido de luz y hacemos zoom hacia ella
  setTimeout(() => {
    if (lunaEl.value) lunaEl.value.classList.add('recibiendo')
  }, ultimo - 250)

  setTimeout(() => emit('next'), ultimo + 1250)
}

function volarHaciaLaLuna(el, delay) {
  const luna = lunaEl.value.getBoundingClientRect()
  const r = el.getBoundingClientRect()
  const dx = luna.left + luna.width / 2 - (r.left + r.width / 2)
  const dy = luna.top + luna.height / 2 - (r.top + r.height / 2)
  const lado = dx > 0 ? 1 : -1
  const duracion = 2400

  // Fijar la posición actual del vagabundeo antes de soltar la animación,
  // para que la luciérnaga despegue desde donde está y no salte
  const actual = getComputedStyle(el).translate
  el.style.animation = 'none'
  el.style.transition = 'none'
  if (actual && actual !== 'none' && actual !== 'auto') el.style.translate = actual
  el.animate(
    [
      { transform: 'translate(0, 0) scale(1)', opacity: 1, offset: 0 },
      { transform: `translate(${dx * 0.45 + lado * 70}px, ${dy * 0.5 - 100}px) scale(1.2)`, opacity: 1, offset: 0.5 },
      { transform: `translate(${dx * 0.8 + lado * 18}px, ${dy * 0.85 + 10}px) scale(0.8)`, opacity: 1, offset: 0.82 },
      { transform: `translate(${dx}px, ${dy}px) scale(0.15)`, opacity: 0, offset: 1 },
    ],
    { duration: duracion, delay, easing: 'cubic-bezier(0.45, 0.05, 0.35, 1)', fill: 'forwards' }
  )
  return delay + duracion
}
</script>

<template>
  <section class="intro" :class="{ volando }" @click="tocar" @touchstart.passive="tocar">
    <!-- ══ Noche del bayou ══ -->
    <div class="stars" aria-hidden="true">
      <span v-for="(s, i) in stars" :key="i" class="star" :style="s"></span>
    </div>

    <div ref="lunaEl" class="luna" aria-hidden="true">
      <span class="crater c1"></span>
      <span class="crater c2"></span>
      <span class="crater c3"></span>
    </div>

    <svg class="bayou" viewBox="0 0 400 720" preserveAspectRatio="xMidYMax slice" aria-hidden="true">
      <defs>
        <linearGradient id="agua" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" stop-color="#101a3e" />
          <stop offset="100%" stop-color="#060a1e" />
        </linearGradient>
        <linearGradient id="niebla" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" stop-color="#8fa8d8" stop-opacity="0" />
          <stop offset="100%" stop-color="#8fa8d8" stop-opacity="0.16" />
        </linearGradient>
      </defs>

      <!-- Agua del bayou -->
      <rect x="0" y="560" width="400" height="160" fill="url(#agua)" />
      <!-- Reflejo de la luna -->
      <ellipse cx="200" cy="575" rx="26" ry="5" fill="#f3e7c0" opacity="0.35" />
      <ellipse cx="200" cy="592" rx="18" ry="3.5" fill="#f3e7c0" opacity="0.22" />
      <ellipse cx="200" cy="608" rx="10" ry="2.5" fill="#f3e7c0" opacity="0.14" />

      <!-- Niebla baja -->
      <rect x="0" y="480" width="400" height="100" fill="url(#niebla)" />

      <!-- Copa del árbol izquierdo con musgo colgante -->
      <path d="M-20 560 Q 30 400 95 370 Q 60 470 110 560 Z" fill="#0a1128" />
      <path d="M20 420 Q 12 470 16 510" stroke="#0a1128" stroke-width="3" fill="none" opacity="0.9" />
      <path d="M48 402 Q 44 455 50 505" stroke="#0a1128" stroke-width="2.5" fill="none" opacity="0.85" />
      <path d="M75 395 Q 74 450 82 500" stroke="#0a1128" stroke-width="2" fill="none" opacity="0.8" />

      <!-- Vegetación derecha -->
      <path d="M420 560 Q 350 420 285 400 Q 330 480 280 560 Z" fill="#0a1128" />
      <path d="M330 430 Q 340 480 336 530" stroke="#0a1128" stroke-width="3" fill="none" opacity="0.9" />
      <path d="M305 415 Q 300 465 306 515" stroke="#0a1128" stroke-width="2.5" fill="none" opacity="0.85" />

      <!-- Juncos -->
      <path d="M60 600 Q 56 560 62 535" stroke="#0b1230" stroke-width="3" fill="none" stroke-linecap="round" />
      <path d="M72 604 Q 70 566 78 545" stroke="#0b1230" stroke-width="2.5" fill="none" stroke-linecap="round" />
      <path d="M340 602 Q 346 562 340 540" stroke="#0b1230" stroke-width="3" fill="none" stroke-linecap="round" />
      <path d="M326 606 Q 324 570 318 550" stroke="#0b1230" stroke-width="2.5" fill="none" stroke-linecap="round" />
    </svg>

    <!-- ══ Las dos luciérnagas ══ -->
    <div class="luci l1" :ref="(el) => setLuci(el, 0)" aria-hidden="true">
      <span class="alas"><i></i><i></i></span>
      <span class="cuerpo"></span>
      <span class="halo"></span>
    </div>
    <div class="luci l2" :ref="(el) => setLuci(el, 1)" aria-hidden="true">
      <span class="alas"><i></i><i></i></span>
      <span class="cuerpo"></span>
      <span class="halo"></span>
    </div>

    <!-- ══ Invitación ══ -->
    <div class="content">
      <h1 class="invite script-title">Agárrame de la mano,</h1>
      <p class="invite-sub">aquí empieza nuestra historia</p>
      <p class="tap-hint">toca la pantalla&nbsp;✦</p>
    </div>
  </section>
</template>

<style scoped>
.intro {
  position: fixed;
  inset: 0;
  z-index: 50;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  overflow: hidden;
  cursor: pointer;
  background:
    radial-gradient(ellipse at 50% 30%, rgba(90, 110, 190, 0.18) 0%, transparent 55%),
    linear-gradient(180deg, #060a20 0%, #0c1334 40%, #16204a 68%, #101a3e 100%);
  /* Zoom final hacia la luna */
  transition:
    transform 1.25s cubic-bezier(0.5, 0, 0.2, 1),
    filter 1.15s ease-in,
    opacity 1.1s ease-in;
  transform-origin: 50% 24%;
}

.intro.volando {
  transform: scale(2.9);
  filter: blur(15px) brightness(1.2);
  opacity: 0;
  pointer-events: none;
}

/* ════════ Cielo ════════ */
.stars {
  position: absolute;
  inset: 0;
}

.star {
  position: absolute;
  border-radius: 50%;
  background: #dfe8ff;
  animation: twinkle ease-in-out infinite;
}

@keyframes twinkle {
  0%, 100% { opacity: 0.12; }
  50% { opacity: 0.9; }
}

/* ════════ Luna ════════ */
.luna {
  position: absolute;
  top: 13%;
  left: 50%;
  transform: translateX(-50%);
  width: clamp(120px, 34vw, 190px);
  aspect-ratio: 1;
  border-radius: 50%;
  background: radial-gradient(circle at 36% 32%, #fffdf2 0%, #f7ecc8 45%, #e8d6a0 100%);
  box-shadow:
    0 0 40px rgba(250, 238, 190, 0.55),
    0 0 130px rgba(250, 238, 190, 0.3),
    0 0 260px rgba(250, 238, 190, 0.15);
  transition: box-shadow 0.9s ease, transform 0.9s ease;
  animation: lunaLatido 7s ease-in-out infinite;
}

.luna.recibiendo {
  box-shadow:
    0 0 60px rgba(255, 246, 205, 0.95),
    0 0 190px rgba(255, 240, 180, 0.55),
    0 0 340px rgba(255, 236, 165, 0.3);
  transform: translateX(-50%) scale(1.08);
}

.crater {
  position: absolute;
  border-radius: 50%;
  background: radial-gradient(circle at 40% 35%, rgba(190, 168, 115, 0.35), rgba(190, 168, 115, 0.12) 70%);
}

.c1 { width: 22%; height: 22%; top: 30%; left: 24%; }
.c2 { width: 14%; height: 14%; top: 58%; left: 55%; }
.c3 { width: 10%; height: 10%; top: 24%; left: 62%; }

@keyframes lunaLatido {
  0%, 100% { box-shadow: 0 0 40px rgba(250, 238, 190, 0.55), 0 0 130px rgba(250, 238, 190, 0.3), 0 0 260px rgba(250, 238, 190, 0.15); }
  50% { box-shadow: 0 0 52px rgba(250, 238, 190, 0.7), 0 0 160px rgba(250, 238, 190, 0.38), 0 0 300px rgba(250, 238, 190, 0.2); }
}

/* ════════ Paisaje ════════ */
.bayou {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
}

/* ════════ Luciérnagas ════════ */
.luci {
  position: absolute;
  width: 14px;
  height: 14px;
  z-index: 3;
  will-change: transform;
}

.l1 { left: 26%; top: 55%; animation: vagar1 9s ease-in-out infinite; }
.l2 { left: 68%; top: 62%; animation: vagar2 11s ease-in-out infinite; }

.cuerpo {
  position: absolute;
  inset: 4px;
  border-radius: 50%;
  background: radial-gradient(circle at 45% 40%, #fffbe8 0%, #ffe9a0 45%, #e8b840 100%);
  box-shadow:
    0 0 8px 2px rgba(255, 238, 170, 0.95),
    0 0 22px 6px rgba(255, 224, 130, 0.55),
    0 0 48px 14px rgba(255, 210, 100, 0.22);
  animation: parpadeo 2.8s ease-in-out infinite;
}

.l2 .cuerpo { animation-delay: 1.3s; }

.halo {
  position: absolute;
  inset: -16px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(255, 232, 150, 0.28) 0%, transparent 70%);
  animation: parpadeo 2.8s ease-in-out infinite;
}

.l2 .halo { animation-delay: 1.3s; }

.alas {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.alas i {
  position: absolute;
  top: -2px;
  width: 9px;
  height: 5px;
  border-radius: 50% 50% 50% 50%;
  background: rgba(220, 235, 255, 0.5);
  filter: blur(0.6px);
  transform-origin: bottom center;
}

.alas i:first-child { left: -7px; animation: aletearIzq 0.14s linear infinite; }
.alas i:last-child { right: -7px; animation: aletearDer 0.14s linear infinite; animation-delay: 0.07s; }

@keyframes aletearIzq {
  0%, 100% { opacity: 0.75; transform: rotate(-32deg) scaleY(1); }
  50% { opacity: 0.3; transform: rotate(-46deg) scaleY(0.3); }
}

@keyframes aletearDer {
  0%, 100% { opacity: 0.75; transform: rotate(32deg) scaleY(1); }
  50% { opacity: 0.3; transform: rotate(46deg) scaleY(0.3); }
}

@keyframes parpadeo {
  0%, 100% { opacity: 0.55; }
  50% { opacity: 1; }
}

@keyframes vagar1 {
  0%, 100% { translate: 0 0; }
  25% { translate: 34px -46px; }
  50% { translate: -22px -18px; }
  75% { translate: 26px 22px; }
}

@keyframes vagar2 {
  0%, 100% { translate: 0 0; }
  30% { translate: -40px -30px; }
  55% { translate: 18px -52px; }
  80% { translate: -28px 16px; }
}

/* ════════ Invitación ════════ */
.content {
  position: relative;
  z-index: 4;
  text-align: center;
  padding: 0 24px calc(52px + env(safe-area-inset-bottom, 0px));
  transition: opacity 0.6s ease, transform 0.6s ease;
}

.intro.volando .content {
  opacity: 0;
  transform: translateY(18px);
}

.invite {
  margin: 0;
  font-size: clamp(2.1rem, 8vw, 3.1rem);
  line-height: 1.15;
  color: #f7ecc8;
  text-shadow:
    0 0 24px rgba(250, 238, 190, 0.4),
    0 3px 18px rgba(4, 6, 20, 0.8);
}

.invite-sub {
  margin: 12px 0 0;
  font-family: var(--font-script);
  font-size: clamp(1.3rem, 4.5vw, 1.7rem);
  color: #b9c6ee;
  text-shadow: 0 2px 14px rgba(4, 6, 20, 0.8);
}

.tap-hint {
  margin: 26px 0 0;
  display: inline-block;
  font-family: var(--font-script);
  font-size: 1.15rem;
  color: #f7ecc8;
  background: rgba(10, 16, 44, 0.55);
  padding: 6px 18px;
  border-radius: 999px;
  border: 1px solid rgba(247, 236, 200, 0.35);
  animation: latidoHint 2.6s ease-in-out infinite;
}

@keyframes latidoHint {
  0%, 100% { opacity: 0.6; }
  50% { opacity: 1; }
}

/* ════════ Móvil ════════ */
@media (max-width: 640px) {
  .content {
    padding: 0 18px calc(40px + env(safe-area-inset-bottom, 0px));
  }

  .l1 { left: 22%; top: 52%; }
  .l2 { left: 72%; top: 60%; }
}

@media (prefers-reduced-motion: reduce) {
  .luci,
  .cuerpo,
  .halo,
  .alas i,
  .star,
  .tap-hint,
  .luna {
    animation: none !important;
  }

  .intro {
    transition: opacity 0.4s ease;
  }
}
</style>
