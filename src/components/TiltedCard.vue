<script setup>
import { ref, computed, onMounted, onUnmounted, useTemplateRef } from 'vue'

const props = defineProps({
  imageSrc: {
    type: String,
    default: '',
  },
  altText: {
    type: String,
    default: 'Tilted card image',
  },
  captionText: {
    type: String,
    default: '',
  },
  containerHeight: {
    type: String,
    default: 'auto',
  },
  containerWidth: {
    type: String,
    default: 'auto',
  },
  imageHeight: {
    type: String,
    default: 'auto',
  },
  imageWidth: {
    type: String,
    default: 'auto',
  },
  scaleOnHover: {
    type: Number,
    default: 1.08,
  },
  rotateAmplitude: {
    type: Number,
    default: 14,
  },
  showMobileWarning: {
    type: Boolean,
    default: false,
  },
  showTooltip: {
    type: Boolean,
    default: false,
  },
  overlayContent: {
    type: Boolean,
    default: false,
  },
  displayOverlayContent: {
    type: Boolean,
    default: false,
  },
  glare: {
    type: Boolean,
    default: true,
  },
})

const cardRef = useTemplateRef('cardRef')

// Estados de rotación objetivo y actual (para física suave Spring/Lerp)
const targetRotateX = ref(0)
const targetRotateY = ref(0)
const targetScale   = ref(1)

const currentRotateX = ref(0)
const currentRotateY = ref(0)
const currentScale   = ref(1)

// Tooltip / Caption flotante
const xValue = ref(0)
const yValue = ref(0)
const opacityValue = ref(0)
const rotateFigcaption = ref(0)
let lastY = 0

// Brillo especular dinámico (glare)
const glareX = ref(50)
const glareY = ref(50)
const glareOpacity = ref(0)

// Loop de física suave (Spring / Damping a 60-120 FPS)
let rafId = null
const lerpFactor = 0.12 // Factor de suavidad / fluidez

function updatePhysics() {
  currentRotateX.value += (targetRotateX.value - currentRotateX.value) * lerpFactor
  currentRotateY.value += (targetRotateY.value - currentRotateY.value) * lerpFactor
  currentScale.value   += (targetScale.value   - currentScale.value)   * lerpFactor

  rafId = requestAnimationFrame(updatePhysics)
}

// ── Interacción con Ratón (Desktop) ──
function handleMouse(e) {
  if (!cardRef.value) return

  const rect = cardRef.value.getBoundingClientRect()
  const offsetX = e.clientX - rect.left - rect.width / 2
  const offsetY = e.clientY - rect.top - rect.height / 2

  targetRotateX.value = (offsetY / (rect.height / 2)) * -props.rotateAmplitude
  targetRotateY.value = (offsetX / (rect.width / 2)) * props.rotateAmplitude

  // Posición del tooltip
  xValue.value = e.clientX - rect.left
  yValue.value = e.clientY - rect.top

  const velocityY = offsetY - lastY
  rotateFigcaption.value = -velocityY * 0.5
  lastY = offsetY

  // Posición del reflejo de luz
  glareX.value = ((e.clientX - rect.left) / rect.width) * 100
  glareY.value = ((e.clientY - rect.top) / rect.height) * 100
  glareOpacity.value = 0.35
}

function handleMouseEnter() {
  targetScale.value = props.scaleOnHover
  opacityValue.value = 1
  glareOpacity.value = 0.35
}

function handleMouseLeave() {
  opacityValue.value = 0
  targetScale.value = 1
  targetRotateX.value = 0
  targetRotateY.value = 0
  rotateFigcaption.value = 0
  glareOpacity.value = 0
}

// ── Interacción con Giroscopio del Celular (DeviceOrientation) ──
let hasGyro = false

function handleOrientation(e) {
  if (e.beta == null || e.gamma == null) return
  hasGyro = true

  // Ángulo de inclinación natural al sostener el celular (~45° frente a los ojos)
  const neutralBeta = 45

  // Normalizar inclinación vertical (beta: hacia adelante/atrás)
  const deltaBeta = e.beta - neutralBeta
  const clampedBeta = Math.max(-30, Math.min(30, deltaBeta))
  targetRotateX.value = (clampedBeta / 30) * -props.rotateAmplitude

  // Normalizar inclinación horizontal (gamma: hacia izquierda/derecha)
  const clampedGamma = Math.max(-30, Math.min(30, e.gamma))
  targetRotateY.value = (clampedGamma / 30) * props.rotateAmplitude

  // Mover el brillo en el celular según la luz virtual
  glareX.value = 50 + (clampedGamma / 30) * 40
  glareY.value = 50 + (clampedBeta / 30) * 40
  glareOpacity.value = 0.4
}

// Solicitar permiso en iOS si es requerido
function requestOrientationPermission() {
  if (
    typeof DeviceOrientationEvent !== 'undefined' &&
    typeof DeviceOrientationEvent.requestPermission === 'function'
  ) {
    DeviceOrientationEvent.requestPermission()
      .then((state) => {
        if (state === 'granted') {
          window.addEventListener('deviceorientation', handleOrientation, { passive: true })
        }
      })
      .catch(() => {})
  }
}

onMounted(() => {
  rafId = requestAnimationFrame(updatePhysics)

  // Escuchar giroscopio en Android / navegadores estándar
  window.addEventListener('deviceorientation', handleOrientation, { passive: true })

  // Para iOS Safari, se habilita al primer toque de pantalla
  window.addEventListener('touchstart', requestOrientationPermission, { once: true, passive: true })
  window.addEventListener('click', requestOrientationPermission, { once: true, passive: true })
})

onUnmounted(() => {
  if (rafId) cancelAnimationFrame(rafId)
  window.removeEventListener('deviceorientation', handleOrientation)
  window.removeEventListener('touchstart', requestOrientationPermission)
  window.removeEventListener('click', requestOrientationPermission)
})

const cardTransform = computed(() => {
  return `perspective(900px) rotateX(${currentRotateX.value.toFixed(2)}deg) rotateY(${currentRotateY.value.toFixed(2)}deg) scale3d(${currentScale.value.toFixed(3)}, ${currentScale.value.toFixed(3)}, 1)`
})
</script>

<template>
  <figure
    ref="cardRef"
    class="tilted-card-container"
    :style="{
      height: containerHeight,
      width: containerWidth,
    }"
    @mousemove="handleMouse"
    @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave"
  >
    <!-- Elemento que rota en 3D con física suave -->
    <div
      class="tilted-card-inner"
      :style="{
        width: imageWidth,
        height: imageHeight,
        transform: cardTransform,
      }"
    >
      <!-- Si se pasa imagen directa -->
      <img
        v-if="imageSrc"
        :src="imageSrc"
        :alt="altText"
        class="tilted-image"
        :style="{
          width: imageWidth,
          height: imageHeight,
        }"
      />

      <!-- Si se pasa contenido personalizado por slot (ej. polaroid, cinta, placeholder) -->
      <div v-else class="tilted-slot-wrap">
        <slot />
      </div>

      <!-- Efecto de brillo especular tipo holográfico (Glare) -->
      <div
        v-if="glare"
        class="tilted-glare"
        :style="{
          background: `radial-gradient(circle at ${glareX}% ${glareY}%, rgba(255, 255, 255, 0.45) 0%, rgba(255, 255, 255, 0.1) 40%, transparent 70%)`,
          opacity: glareOpacity,
        }"
        aria-hidden="true"
      />

      <!-- Overlay flotante en 3D (con elevación en Z) -->
      <div
        v-if="displayOverlayContent"
        class="tilted-overlay"
      >
        <slot name="overlay" />
      </div>
    </div>

    <!-- Tooltip con seguimiento suave -->
    <figcaption
      v-if="showTooltip && captionText"
      class="tilted-tooltip"
      :style="{
        transform: `translate3d(${xValue}px, ${yValue}px, 0) rotate(${rotateFigcaption}deg)`,
        opacity: opacityValue,
      }"
    >
      {{ captionText }}
    </figcaption>
  </figure>
</template>

<style scoped>
.tilted-card-container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  perspective: 900px;
  margin: 0;
  padding: 0;
  cursor: pointer;
  touch-action: pan-y;
}

.tilted-card-inner {
  position: relative;
  transform-style: preserve-3d;
  will-change: transform;
  display: flex;
  align-items: center;
  justify-content: center;
}

.tilted-image {
  position: relative;
  display: block;
  object-fit: cover;
  border-radius: 12px;
  transform: translateZ(0);
  will-change: transform;
  box-shadow: 0 15px 35px rgba(74, 15, 30, 0.25);
}

.tilted-slot-wrap {
  position: relative;
  width: 100%;
  height: 100%;
  transform-style: preserve-3d;
}

/* Efecto de reflejo holográfico al inclinar */
.tilted-glare {
  position: absolute;
  inset: 0;
  border-radius: inherit;
  pointer-events: none;
  mix-blend-mode: overlay;
  transition: opacity 0.3s ease;
  z-index: 2;
  transform: translateZ(1px);
}

/* Capa de contenido flotante con profundidad 3D */
.tilted-overlay {
  position: absolute;
  inset: 0;
  z-index: 3;
  transform: translateZ(28px);
  display: flex;
  align-items: center;
  justify-content: center;
  pointer-events: none;
}

/* Tooltip elegante flotante */
.tilted-tooltip {
  position: absolute;
  left: 0;
  top: 0;
  pointer-events: none;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.94);
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
  padding: 5px 12px;
  font-family: var(--font-serif, sans-serif);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.04em;
  color: #2d2d2d;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  z-index: 10;
  transition: opacity 0.25s ease;
  white-space: nowrap;
}

@media (max-width: 600px) {
  .tilted-tooltip {
    display: none; /* En celular el giro del giroscopio es el protagonista */
  }
}
</style>
