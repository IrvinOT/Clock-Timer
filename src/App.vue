<template>
  <div class="time-display">{{ timeInSeconds }}s</div>
  <div class="container" ref="container">
    <div class="clock" ref="clock">
      <div class="hand second" :style="{ transform: `rotate(${second}deg)` }"
        @mousedown="startDragHand('second', $event)"></div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

const clock = ref(null);
const second = ref(0);
let maxMinuet = ref(60)
let status = ref({
  draggin: false,
})
let draggingHand = null;
let startAngle = 0;
let rewindValue = 360 / maxMinuet.value

const timeInSeconds = computed(() => Math.round((second.value / 360) * maxMinuet.value));

const startDragHand = (hand, event) => {
  draggingHand = hand;
  startAngle = getAngle(event.clientX, event.clientY);
  window.addEventListener('mousemove', dragHand);
  window.addEventListener('mouseup', stopDragHand);
};


const dragHand = (event) => {
  if (!draggingHand) return;
  status.value.draggin = true
  let currentAngle = getAngle(event.clientX, event.clientY);
  const angleDifference = currentAngle - startAngle;
  second.value = normalizeAngle(second.value + angleDifference);
  startAngle = currentAngle;
};

const stopDragHand = () => {
  if (draggingHand) adjustSecondHand()
  draggingHand = null;
  status.value.draggin = false
  window.removeEventListener('mousemove', dragHand);
  window.removeEventListener('mouseup', stopDragHand);
};

const adjustSecondHand = () =>{
  const secondsFromAngle = Math.round((second.value / 360) * maxMinuet.value);
  second.value = (secondsFromAngle / maxMinuet.value) * 360; // Mapea el segundo al ángulo correspondiente
}

const normalizeAngle = (angle) =>  (angle + 360) % 360

const getAngle = (ex, ey) => {
  let clockRect = clock.value.getBoundingClientRect();
  let centerX = clockRect.left + clockRect.width / 2;
  let centerY = clockRect.top + clockRect.height / 2;
  let dy = ey - centerY;
  let dx = ex - centerX;
  let theta = Math.atan2(dy, dx); // range (-PI, PI)
  theta *= 180 / Math.PI; // rads to degs, range (-180, 180)
  return theta;
};

onMounted(() => {
  setInterval(() => {
    if(status.value.draggin) return
    let newAngle = second.value - rewindValue
    second.value = newAngle > 0 ? newAngle : 0
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
  background-color: #eee;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.hand {
  position: absolute;
  background-color: black;
  transform-origin: 50% 100%;
  left: 50%;
  cursor: grab;
  /* Indicate that the hand is draggable */
}

.hand:active {
  cursor: grabbing;
  /* Indicate that the hand is being dragged */
}

.second {
  width: 2px;
  height: 240px;
  background-color: red;
  top: 3%;
  left: 50%;
}

.time-display {
  margin-top: 20px;
  font-size: 1.5rem;
  font-weight: bold;
  text-align: center;
  color: white;
}
</style>
