<template>
  <div class="clock-container">
    <ClientOnly>
      <div class="clock-display">
        {{ currentTime }}
      </div>
    </ClientOnly>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const currentTime = ref('')

const updateTime = () => {
  const now = new Date()
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  const seconds = String(now.getSeconds()).padStart(2, '0')
  currentTime.value = `${hours}:${minutes}:${seconds}`
}

let timer: ReturnType<typeof setInterval>

onMounted(() => {
  updateTime()
  timer = setInterval(updateTime, 1000)
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
})

useHead({
  title: 'Digital Clock',
  bodyAttrs: {
    style: 'margin: 0; background-color: #000; overflow: hidden;'
  }
})
</script>

<style scoped>
.clock-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100vw;
  background: radial-gradient(circle, #1a1a1a 0%, #000 100%);
  color: #fff;
  font-family: 'JetBrains Mono', monospace;
  overflow: hidden;
}

.clock-display {
  font-size: 18vw; /* Adjusted to fit HH:MM:SS on screen */
  font-weight: 700;
  line-height: 1;
  text-shadow: 0 0 30px rgba(255, 255, 255, 0.3);
  white-space: nowrap;
}
</style>