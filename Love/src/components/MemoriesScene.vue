<script setup>
import { ref, computed } from 'vue'
import { photos, frases } from '../data/content'

const emit = defineEmits(['next'])

const opened = ref(false)
const photoIdx = ref(0)
const fraseIdx = ref(0)
const failed = ref(new Set())

const photo = computed(() => photos[photoIdx.value])
const frase = computed(() => frases[fraseIdx.value])

// Índices ya usados para no repetir hasta agotar
const usedP = ref(new Set([0]))
const usedF = ref(new Set([0]))

function abrir() {
  opened.value = true
  sorprender()
}

function sorprender() {
  const pick = (len, used) => {
    if (used.value.size >= len) used.value.clear()
    let i
    do {
      i = Math.floor(Math.random() * len)
    } while (used.value.has(i))
    used.value.add(i)
    return i
  }
  photoIdx.value = pick(photos.length, usedP)
  fraseIdx.value = pick(frases.length, usedF)
}

function onImgError(i) {
  failed.value.add(i)
}
</script>

<template>
  <section class="scene">
    <h1 class="script-title head fade-up">Recuerdos al azar</h1>

    <!-- Mini sobre cerrado -->
    <div v-if="!opened" class="mini-envelope-zone fade-up" style="animation-delay: 0.25s">
      <button class="mini-envelope" aria-label="Abrir recuerdos" @click="abrir">
        <span class="mini-seal">♥</span>
      </button>
      <p class="tap-hint">abre cuando estés lista</p>
    </div>

    <!-- Recuerdo revelado -->
    <div v-else class="memory fade-up">
      <div class="memory-card" :key="photoIdx + '-' + fraseIdx">
        <div class="photo-wrap">
          <img
            v-if="!failed.has(photoIdx)"
            :src="photo.src"
            :alt="photo.caption"
            @error="onImgError(photoIdx)"
          />
          <div v-else class="placeholder" :style="{ background: photo.gradient }">
            <span>Nuestro recuerdo</span>
          </div>
          <span class="tape"></span>
        </div>
        <blockquote class="frase">“{{ frase }}”</blockquote>
      </div>

      <div class="actions">
        <button class="btn-ghost" @click="sorprender">✨ Otro recuerdo</button>
        <button class="btn-love" @click="emit('next')">Una última cosa…&nbsp;💌</button>
      </div>
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
  gap: 36px;
  padding: 48px 20px;
}

.head {
  margin: 0;
  font-size: clamp(2.4rem, 6vw, 3.6rem);
}

.tap-hint {
  margin: 16px 0 0;
  font-style: italic;
  opacity: 0.6;
  color: var(--wine);
}

/* ---------- Mini sobre ---------- */
.mini-envelope-zone {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.mini-envelope {
  position: relative;
  width: min(260px, 70vw);
  aspect-ratio: 260 / 170;
  background: linear-gradient(160deg, #f3d9c9 0%, #e5b89a 100%);
  border: none;
  border-radius: 8px;
  clip-path: polygon(0 0, 100% 0, 50% 52%, 100% 100%, 0 100%);
  filter: drop-shadow(0 14px 28px rgba(74, 15, 30, 0.28));
  transition: transform 0.35s ease;
}

.mini-envelope:hover {
  transform: translateY(-6px) scale(1.03) rotate(1deg);
}

.mini-seal {
  position: absolute;
  left: 50%;
  top: 40%;
  transform: translate(-50%, -50%);
  width: 46px;
  height: 46px;
  display: grid;
  place-items: center;
  font-size: 19px;
  color: #ffd9de;
  background: radial-gradient(circle at 35% 30%, #a92440, #6d1a2d 70%);
  border-radius: 50%;
  box-shadow: 0 4px 10px rgba(74, 15, 30, 0.4);
}

/* ---------- Recuerdo ---------- */
.memory {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 30px;
}

.memory-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 26px;
  animation: popIn 0.7s cubic-bezier(0.22, 1, 0.36, 1) both;
}

@keyframes popIn {
  from {
    opacity: 0;
    transform: scale(0.9) rotate(-1.5deg);
  }
  to {
    opacity: 1;
    transform: scale(1) rotate(0);
  }
}

.photo-wrap {
  position: relative;
  width: min(340px, 82vw);
}

.photo-wrap img,
.placeholder {
  width: 100%;
  aspect-ratio: 4 / 3.4;
  object-fit: cover;
  display: block;
  background: #fff;
  padding: 12px 12px 12px;
  border-radius: 3px;
  box-shadow:
    0 2px 4px rgba(74, 15, 30, 0.12),
    0 18px 40px rgba(74, 15, 30, 0.22);
  transform: rotate(-1.2deg);
}

.placeholder {
  display: grid;
  place-items: center;
  color: rgba(255, 255, 255, 0.92);
  font-family: var(--font-script);
  font-size: 1.3rem;
  text-shadow: 0 1px 4px rgba(0, 0, 0, 0.15);
}

.tape {
  position: absolute;
  top: -12px;
  left: 50%;
  transform: translateX(-50%) rotate(-3deg);
  width: 96px;
  height: 26px;
  background: rgba(232, 164, 180, 0.55);
  border-left: 1px dashed rgba(255, 255, 255, 0.6);
  border-right: 1px dashed rgba(255, 255, 255, 0.6);
  box-shadow: 0 2px 6px rgba(74, 15, 30, 0.12);
}

.frase {
  margin: 0;
  max-width: 46ch;
  text-align: center;
  font-family: var(--font-script);
  font-size: clamp(1.3rem, 3.6vw, 1.7rem);
  line-height: 1.5;
  color: var(--wine);
}

.actions {
  display: flex;
  gap: 18px;
  flex-wrap: wrap;
  justify-content: center;
}

@media (max-width: 640px) {
  .scene {
    gap: 26px;
    padding: 32px 14px 76px;
  }

  .actions {
    flex-direction: column;
    width: 100%;
    max-width: 340px;
  }

  .actions .btn-love,
  .actions .btn-ghost {
    justify-content: center;
  }

  .frase {
    font-size: 1.25rem;
  }
}
</style>
