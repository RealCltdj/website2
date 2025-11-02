<script setup lang="ts">
import { reactive, ref, type Ref } from 'vue';

interface DeviceMotionState {
  acceleration: DeviceMotionEvent['acceleration'] | null;
  accelerationIncludingGravity: DeviceMotionEvent['accelerationIncludingGravity'] | null;
  rotationRate: DeviceMotionEvent['rotationRate'] | null;
  interval: number | null;
}

type PermissionState = "granted" | "denied" | "not_required" | "failed_to_ask" | "asking"

// Initialize reactive state with null values
const ev = reactive<DeviceMotionState>({
  acceleration: null,
  accelerationIncludingGravity: null,
  rotationRate: null,
  interval: null,
});

const permission: Ref<PermissionState> = ref("not_required");
const error = ref("");
const need_to_ask = (typeof DeviceMotionEvent !== 'undefined' && 'requestPermission' in DeviceMotionEvent);

async function requestPermissionAndListen() {
  // iOS requires permission to access device motion events
  if (need_to_ask) {
    try {
      permission.value = "asking";
      const response: PermissionState = await DeviceMotionEvent.requestPermission();
      permission.value = response;
      if (response === 'granted') {
        window.addEventListener('devicemotion', handleMotion);
      } else {
        console.warn('Permission to access device motion denied.');
      }
    } catch (err) {
      permission.value = "failed_to_ask";
      console.error('Error requesting device motion permission:', err);
      if (err instanceof Error) {
        error.value = err.toString();
      }
    }
  } else {
    // Non iOS or no permission required
    window.addEventListener('devicemotion', handleMotion);
  }
}

function handleMotion(event: DeviceMotionEvent) {
  ev.acceleration = event.acceleration;
  ev.accelerationIncludingGravity = event.accelerationIncludingGravity;
  ev.rotationRate = event.rotationRate;
  ev.interval = event.interval;
}

if (!need_to_ask) {
  requestPermissionAndListen();
}
</script>

<template>
  <h1>Device Motion</h1>
  <p>Permission: {{ permission }}</p>
  <button v-if="need_to_ask" @click="requestPermissionAndListen">Grant Access</button>
  <p v-if="error != ''">Error: {{ error }}</p>
  <pre>{{ JSON.stringify(ev, null, 2) }}</pre>
</template>

<style scoped></style>
