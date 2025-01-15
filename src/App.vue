<template>
  <div class="container" ref="container">
    <div class="clock" ref="clock">
      <div class="hand hour" :style="{ transform: `rotate(${hour}deg)` }" @mousedown="startDragHand('hour', $event)"></div>
      <div class="hand minute" :style="{ transform: `rotate(${minute}deg)` }" @mousedown="startDragHand('minute', $event)"></div>
      <div class="hand second" :style="{ transform: `rotate(${second}deg)` }" @mousedown="startDragHand('second', $event)"></div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const clock = ref(null);
const hour = ref(0);
const minute = ref(0);
const second = ref(0);

let draggingHand = null;
let startAngle = 0;
let rewindValue = 360/60

const startDragHand = (hand, event) => {
  draggingHand = hand;
  const clockRect = clock.value.getBoundingClientRect();
  const centerX = clockRect.left + clockRect.width / 2;
  const centerY = clockRect.top + clockRect.height / 2;
  startAngle = getAngle(centerX, centerY, event.clientX, event.clientY);
  window.addEventListener('mousemove', dragHand);
  window.addEventListener('mouseup', stopDragHand);
};

const dragHand = (event) => {
  if (draggingHand) {
    const clockRect = clock.value.getBoundingClientRect();
    const centerX = clockRect.left + clockRect.width / 2;
    const centerY = clockRect.top + clockRect.height / 2;
    const currentAngle = getAngle(centerX, centerY, event.clientX, event.clientY);
    const angleDifference = currentAngle - startAngle;

    switch (draggingHand) {
      case 'hour':
        hour.value = (hour.value + angleDifference) % 360;
        break;
      case 'minute':
        minute.value = (minute.value + angleDifference) % 360;
        break;
      case 'second':
        second.value = (second.value + angleDifference) % 360;
        break;
    }
    startAngle = currentAngle;
  }
};

const stopDragHand = () => {
  draggingHand = null;
  window.removeEventListener('mousemove', dragHand);
  window.removeEventListener('mouseup', stopDragHand);
};

const getAngle = (cx, cy, ex, ey) => {
  const dy = ey - cy;
  const dx = ex - cx;
  let theta = Math.atan2(dy, dx); // range (-PI, PI)
  theta *= 180 / Math.PI; // rads to degs, range (-180, 180)
  return theta;
};

onMounted(() => {
  setInterval(() => {
    const now = new Date();
    hour.value = (now.getHours() % 12 + now.getMinutes() / 60) * 30;
    minute.value = now.getMinutes() * rewindValue;
    second.value = now.getSeconds() * rewindValue;
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
  cursor: grab; /* Indicate that the hand is draggable */
}

.hand:active {
  cursor: grabbing; /* Indicate that the hand is being dragged */
}


.hour {
  width: 4px;
  height: 220px;
  top: 6%;
  left: 49.3%;
}

.minute {
  width: 3px;
  height: 235px;
  top: 3%;
  left: 49.5%;
}

.second {
  width: 2px;
  height: 250px;
  background-color: red;
  left: 49.5%;
}
</style>
