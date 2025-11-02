<script setup lang="ts">
import { reactive } from 'vue';

let ev = reactive({});

try {
  DeviceMotionEvent.requestPermission().then(response => {
    if (response === 'granted') {
      // You can now listen to motion events
    }
  });
}
finally {
  
}

window.addEventListener("devicemotion", (event) => {
  ev = event;
})

</script>

<template>
  <h1>devicemotion</h1>
  <pre>
    {{ JSON.stringify(ev) }}
  </pre>
</template>

<style scoped></style>
