<script setup>
import { ref, onMounted } from 'vue'
import { clausulas } from '../data/content'
import { config } from '../data/config'

const emit = defineEmits(['next'])

const canvas = ref(null)
const firmado = ref(false)
const drawing = ref(false)
const lastPoint = ref(null)

onMounted(() => {
  const ctx = canvas.value.getContext('2d')
  ctx.lineWidth = 2.2
  ctx.lineCap = 'round'
  ctx.lineJoin = 'round'
  ctx.strokeStyle = '#4a0f1e'
})

function getPoint(e) {
  const rect = canvas.value.getBoundingClientRect()
  return {
    x: ((e.clientX - rect.left) / rect.width) * canvas.value.width,
    y: ((e.clientY - rect.top) / rect.height) * canvas.value.height,
  }
}

function startDraw(e) {
  drawing.value = true
  lastPoint.value = getPoint(e)
}

function moveDraw(e) {
  if (!drawing.value) return
  e.preventDefault()
  const ctx = canvas.value.getContext('2d')
  const p = getPoint(e)
  ctx.beginPath()
  ctx.moveTo(lastPoint.value.x, lastPoint.value.y)
  ctx.lineTo(p.x, p.y)
  ctx.stroke()
  lastPoint.value = p
  if (!firmado.value) firmado.value = true
}

function endDraw() {
  drawing.value = false
}

function limpiar() {
  const c = canvas.value
  c.getContext('2d').clearRect(0, 0, c.width, c.height)
  firmado.value = false
}

function sellar() {
  emit('next')
}
</script>

<template>
  <section class="scene">
    <div class="paper contract fade-up">
      <header class="head">
        <div class="ornament">❦</div>
        <h1 class="script-title">Contrato de Amor Eterno</h1>
        <p class="meta">
          Celebrado entre <strong>{{ config.herName }}</strong> y
          <strong>{{ config.yourName }}</strong>, en el día de su aniversario,
          ante el testigo silencioso del tiempo.
        </p>
      </header>

      <ol class="clauses">
        <li v-for="(c, i) in clausulas" :key="i" class="clause">
          <span class="emoji">{{ c.emoji }}</span>
          <span><em>Cláusula {{ i + 1 }}.</em> {{ c.texto }}</span>
        </li>
      </ol>

      <p class="acceptance">
        Quien firme este documento acepta amar sin condiciones,
        reír sin miedo y construir recuerdos sin fin.
      </p>

      <!-- Zona de firma -->
      <div class="sign-zone">
        <div class="sign-box">
          <canvas
            ref="canvas"
            width="520"
            height="170"
            @pointerdown="startDraw"
            @pointermove="moveDraw"
            @pointerup="endDraw"
            @pointerleave="endDraw"
          />
          <span v-if="!firmado" class="sign-hint">firma aquí con el dedo o el mouse ✍️</span>
          <span class="sign-line"></span>
          <span class="sign-label">Firma de {{ config.herName }}</span>
        </div>

        <div class="sign-actions">
          <button class="btn-ghost" :disabled="!firmado" @click="limpiar">Borrar</button>
          <button class="btn-love" :disabled="!firmado" @click="sellar">
            Sellar nuestro contrato&nbsp;💗
          </button>
        </div>
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
  align-items: center;
  justify-content: center;
  padding: 48px 18px;
}

.contract {
  width: min(680px, 100%);
  padding: clamp(28px, 5vw, 52px);
  text-align: center;
}

.ornament {
  font-size: 1.6rem;
  color: var(--gold);
  margin-bottom: 4px;
}

h1 {
  margin: 0;
  font-size: clamp(2.2rem, 6vw, 3.2rem);
}

.meta {
  margin: 14px auto 0;
  max-width: 46ch;
  font-style: italic;
  font-size: clamp(0.95rem, 2.5vw, 1.05rem);
  color: var(--ink);
  opacity: 0.8;
  line-height: 1.5;
}

.clauses {
  list-style: none;
  margin: 30px 0 0;
  padding: 0;
  text-align: left;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.clause {
  display: flex;
  gap: 14px;
  align-items: baseline;
  font-size: clamp(1rem, 2.6vw, 1.15rem);
  line-height: 1.45;
  padding-bottom: 14px;
  border-bottom: 1px dashed rgba(201, 162, 75, 0.4);
}

.clause:last-child {
  border-bottom: none;
}

.clause .emoji {
  font-size: 1.25rem;
}

.clause em {
  color: var(--wine);
  font-weight: 600;
}

.acceptance {
  margin: 26px 0 0;
  font-family: var(--font-script);
  font-size: clamp(1.2rem, 3vw, 1.5rem);
  color: var(--wine);
  line-height: 1.4;
}

/* ---------- Firma ---------- */
.sign-zone {
  margin-top: 34px;
}

.sign-box {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
}

canvas {
  width: min(520px, 100%);
  height: 150px;
  touch-action: none;
  cursor: crosshair;
  background:
    repeating-linear-gradient(
      to bottom,
      transparent 0 34px,
      rgba(109, 26, 45, 0.06) 34px 35px
    );
  border-radius: 6px;
}

.sign-hint {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-style: italic;
  color: var(--ink);
  opacity: 0.4;
  pointer-events: none;
  font-size: 0.98rem;
  transition: opacity 0.3s ease;
}

.sign-line {
  width: min(480px, 92%);
  height: 1.5px;
  margin-top: -6px;
  background: linear-gradient(90deg, transparent, var(--wine), transparent);
  opacity: 0.5;
}

.sign-label {
  margin-top: 10px;
  font-family: var(--font-script);
  font-size: 1.2rem;
  color: var(--ink);
  opacity: 0.75;
}

.sign-actions {
  margin-top: 22px;
  display: flex;
  gap: 16px;
  justify-content: center;
  flex-wrap: wrap;
}

@media (max-width: 640px) {
  .scene {
    padding: 28px 12px 76px;
    align-items: flex-start;
  }

  .contract {
    padding: 26px 18px;
  }

  .clauses {
    gap: 13px;
    margin-top: 22px;
  }

  .clause {
    font-size: 0.98rem;
    gap: 10px;
    padding-bottom: 11px;
  }

  canvas {
    height: 130px;
  }

  .sign-actions {
    flex-direction: column;
    align-items: stretch;
  }

  .sign-actions .btn-love,
  .sign-actions .btn-ghost {
    justify-content: center;
    padding: 13px 20px;
    font-size: 1.05rem;
  }
}
</style>
