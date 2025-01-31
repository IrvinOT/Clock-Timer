<template>
  <div class="time-display">{{ timeInSeconds }}s</div>
  <div class="container" ref="container">
    <div class="clock" ref="clock" :style="{ background: dynamicBackground }" :class="{ dragging: status.draggin }">
      <div class="hand second" :style="{ transform: `rotate(${second}deg)` }"
        @mousedown="startDragHand('second', $event)"></div>
    </div>
    <div v-if="status.draggin" class="floating-time" :style="floatingTimeStyle">
      {{ timeInSeconds }}s
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue';

const clock = ref(null);
const second = ref(0);
const maxSeconds = ref(60);
const status = ref({
  draggin: false,
});
const dynamicBackground = ref('');
const draggingHand = ref(null);
const startAngle = ref(0);
const cursorPosition = ref({ x: 0, y: 0 });
const rewindValue = 360 / maxSeconds.value;

// Tiempo calculado desde el ángulo actual
const timeInSeconds = computed(() => Math.round((second.value / 360) * maxSeconds.value));

const updateBackground = () => {
  const activeAngle = (second.value / 360) * 100; // Porcentaje del gradiente activo
  dynamicBackground.value = `conic-gradient(
    #4FB3F5 0% ${activeAngle}%,  
    #E8E8E8 ${activeAngle}% 100%    
  )`;
};
watch(second, updateBackground);

// Estilo para el marcador flotante
const floatingTimeStyle = computed(() => ({
  top: `${cursorPosition.value.y - 25}px`,
  left: `${cursorPosition.value.x}px`,
}));

// Manejo del arrastre
const startDragHand = (hand, event) => {
  draggingHand.value = hand;
  startAngle.value = getAngle(event.clientX, event.clientY);
  status.value.draggin = true;
  cursorPosition.value = { x: event.clientX, y: event.clientY };
  window.addEventListener('mousemove', dragHand);
  window.addEventListener('mouseup', stopDragHand);
};

const dragHand = (event) => {
  if (!draggingHand.value) return;
  cursorPosition.value = { x: event.clientX, y: event.clientY };

  const currentAngle = getAngle(event.clientX, event.clientY);
  const angleDifference = currentAngle - startAngle.value;
  second.value = normalizeAngle(second.value + angleDifference);
  startAngle.value = currentAngle;
};

const stopDragHand = () => {
  if (draggingHand.value) adjustSecondHand();
  draggingHand.value = null;
  status.value.draggin = false;
  window.removeEventListener('mousemove', dragHand);
  window.removeEventListener('mouseup', stopDragHand);
};

// Ajusta el ángulo al segundo más cercano
const adjustSecondHand = () => {
  const secondsFromAngle = Math.round((second.value / 360) * maxSeconds.value);
  second.value = (secondsFromAngle / maxSeconds.value) * 360;
};

// Normaliza un ángulo entre 0 y 360 grados
const normalizeAngle = (angle) => (angle + 360) % 360;

// Calcula el ángulo entre el centro del reloj y el puntero
const getAngle = (ex, ey) => {
  const clockRect = clock.value.getBoundingClientRect();
  const centerX = clockRect.left + clockRect.width / 2;
  const centerY = clockRect.top + clockRect.height / 2;
  const dy = ey - centerY;
  const dx = ex - centerX;
  const theta = Math.atan2(dy, dx) * (180 / Math.PI); // Convierte de radianes a grados
  return theta;
};

onMounted(() => {
  setInterval(() => {
    if (status.value.draggin) return;
    let newAngle = second.value - rewindValue;
    second.value = newAngle > 0 ? newAngle : 0;
  }, 1000);
});
</script>

<style scoped>
.container {
  width: 500px;
  height: 500px;
  position: relative;
}

.clock {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: #E8E8E8;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  box-shadow: inset 0 4px 10px rgba(0, 0, 0, 0.5), inset 0 -4px 10px rgba(255, 255, 255, 0.5);
  overflow: hidden;
}


.clock.dragging {
  background-color: #4FB3F5;
  box-shadow: inset 0 4px 10px rgba(0, 0, 0, 0.2), inset 0 -4px 10px rgba(255, 255, 255, 0.5), 0 12px 25px #D3D3D3;
}

.hand {
  position: absolute;
  background-color: #4FB3F5;
  transform-origin: 50% 100%;
  left: 50%;
  width: 2px;
  height: 50%;
  z-index: 3;
  border-radius: 2px;
  box-shadow: 0 0 3px #4FB3F5;
  cursor: grab;
}

.hand:active {
  cursor: grabbing;
}



.floating-time {
  position: fixed;
  background-color: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  font-size: 14px;
  pointer-events: none;
  transform: translate(-50%, -50%);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
}
</style>
