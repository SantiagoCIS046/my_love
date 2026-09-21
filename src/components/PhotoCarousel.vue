<script setup>
import { ref } from 'vue'
import { photos } from '../data/content'

const emit = defineEmits(['next'])

const track = ref(null)
const failed = ref(new Set())

function onImgError(i) {
  failed.value.add(i)
}

function scrollBy(dir) {
  const el = track.value
  if (!el) return
  el.scrollBy({ left: dir * Math.min(el.clientWidth * 0.8, 420), behavior: 'smooth' })
}
</script>

<template>
  <section class="scene">
    <header class="head fade-up">
      <h1 class="script-title">Nuestros momentos</h1>
      <p class="sub">Desliza y recorre nuestro álbum &nbsp;💌</p>
    </header>

    <div class="carousel-zone fade-up" style="animation-delay: 0.25s">
      <button class="arrow left" aria-label="Anterior" @click="scrollBy(-1)">‹</button>

      <div ref="track" class="track">
        <article
          v-for="(p, i) in photos"
          :key="i"
          class="polaroid"
          :style="{ '--tilt': (i % 2 === 0 ? -1 : 1) * (1.5 + (i % 3)) + 'deg' }"
        >
          <div class="photo-frame">
            <img
              v-if="!failed.has(i)"
              :src="p.src"
              :alt="p.caption"
              loading="lazy"
              @error="onImgError(i)"
            />
            <div v-else class="placeholder" :style="{ background: p.gradient }">
              <span>Aquí va<br />nuestra foto</span>
            </div>
          </div>
          <p class="caption">{{ p.caption }}</p>
        </article>
      </div>

      <button class="arrow right" aria-label="Siguiente" @click="scrollBy(1)">›</button>
    </div>

    <div class="actions fade-up" style="animation-delay: 0.5s">
      <button class="btn-love" @click="emit('next')">
        Continuar &nbsp;→
      </button>
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
  gap: 40px;
  padding: 48px 16px;
}

.head {
  text-align: center;
}

h1 {
  margin: 0;
  font-size: clamp(2.6rem, 7vw, 4.2rem);
}

.sub {
  margin: 8px 0 0;
  font-family: var(--font-script);
  font-size: clamp(1.2rem, 3vw, 1.6rem);
  color: var(--ink);
  opacity: 0.75;
}

/* ---------- Carrusel ---------- */
.carousel-zone {
  position: relative;
  width: min(1100px, 100%);
  display: flex;
  align-items: center;
}

.track {
  display: flex;
  gap: 26px;
  overflow-x: auto;
  overflow-y: hidden;
  scroll-snap-type: x mandatory;
  -webkit-overflow-scrolling: touch;
  touch-action: pan-x pan-y;
  padding: 30px 8vw 34px;
  scrollbar-width: none;
  cursor: grab;
}

.track::-webkit-scrollbar {
  display: none;
}

.track:active {
  cursor: grabbing;
}

.polaroid {
  scroll-snap-align: center;
  flex: 0 0 auto;
  width: clamp(220px, 26vw, 280px);
  background: #fff;
  padding: 14px 14px 18px;
  border-radius: 4px;
  box-shadow:
    0 2px 4px rgba(74, 15, 30, 0.12),
    0 16px 34px rgba(74, 15, 30, 0.18);
  transform: rotate(var(--tilt));
  transition: transform 0.35s cubic-bezier(0.22, 1, 0.36, 1), box-shadow 0.35s ease;
}

.polaroid:hover {
  transform: rotate(0deg) translateY(-10px) scale(1.04);
  box-shadow:
    0 4px 8px rgba(74, 15, 30, 0.15),
    0 26px 54px rgba(74, 15, 30, 0.28);
  z-index: 2;
}

.photo-frame {
  aspect-ratio: 4 / 4.4;
  border-radius: 2px;
  overflow: hidden;
  background: var(--blush);
}

.photo-frame img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.placeholder {
  width: 100%;
  height: 100%;
  display: grid;
  place-items: center;
  text-align: center;
  color: rgba(255, 255, 255, 0.92);
  font-family: var(--font-script);
  font-size: 1.35rem;
  line-height: 1.3;
  text-shadow: 0 1px 4px rgba(0, 0, 0, 0.15);
}

.caption {
  margin: 14px 4px 0;
  text-align: center;
  font-family: var(--font-script);
  font-size: 1.15rem;
  color: var(--ink);
}

/* ---------- Flechas ---------- */
.arrow {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  z-index: 3;
  width: 48px;
  height: 48px;
  display: grid;
  place-items: center;
  font-size: 30px;
  line-height: 1;
  color: var(--wine);
  background: rgba(255, 250, 243, 0.9);
  border: 1px solid rgba(109, 26, 45, 0.18);
  border-radius: 50%;
  box-shadow: 0 4px 14px rgba(74, 15, 30, 0.18);
  transition: transform 0.25s ease, background 0.25s ease;
}

.arrow:hover {
  background: #fff;
  transform: translateY(-50%) scale(1.1);
}

.arrow.left { left: -6px; }
.arrow.right { right: -6px; }

@media (max-width: 640px) {
  .scene {
    gap: 26px;
    padding: 32px 10px;
  }

  .arrow { display: none; }

  .track {
    gap: 18px;
    padding: 20px 12vw 26px;
  }

  .polaroid {
    width: min(250px, 68vw);
  }

  .actions {
    padding-bottom: 56px; /* espacio para la barra de progreso */
  }
}
</style>
