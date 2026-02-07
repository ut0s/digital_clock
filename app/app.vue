<template>
  <div class="clock-container">
    <button class="fullscreen-button" @click="toggleFullscreen" :title="isFullscreen ? 'Exit Fullscreen' : 'Enter Fullscreen'">
      <svg v-if="!isFullscreen" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
        <path d="M7 14H5v5h5v-2H7v-3zm-2-4h2V7h3V5H5v5zm12 7h-3v2h5v-5h-2v3zM14 5v2h3v3h2V5h-5z"/>
      </svg>
      <svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
        <path d="M5 16h3v3h2v-5H5v2zm3-8H5v2h5V5H8v3zm6 11h2v-3h3v-2h-5v5zm2-11V5h-2v5h5V8h-3z"/>
      </svg>
    </button>
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
const isFullscreen = ref(false)
let wakeLock: WakeLockSentinel | null = null

const updateTime = () => {
  const now = new Date()
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  const seconds = String(now.getSeconds()).padStart(2, '0')
  currentTime.value = `${hours}:${minutes}:${seconds}`
}

const requestWakeLock = async () => {
  if ('wakeLock' in navigator) {
    try {
      wakeLock = await navigator.wakeLock.request('screen')
      wakeLock.addEventListener('release', () => {
        console.log('Wake Lock was released')
      })
    } catch (err: any) {
      console.error(`${err.name}, ${err.message}`)
    }
  }
}

const handleVisibilityChange = async () => {
  if (wakeLock !== null && document.visibilityState === 'visible') {
    await requestWakeLock()
  }
}

const toggleFullscreen = () => {
  if (!document.fullscreenElement) {
    document.documentElement.requestFullscreen().catch(err => {
      console.error(`Error attempting to enable full-screen mode: ${err.message} (${err.name})`)
    })
  } else {
    document.exitFullscreen()
  }
}

const handleFullscreenChange = () => {
  isFullscreen.value = !!document.fullscreenElement
}

let timer: ReturnType<typeof setInterval>

onMounted(async () => {
  updateTime()
  timer = setInterval(updateTime, 1000)
  document.addEventListener('fullscreenchange', handleFullscreenChange)
  document.addEventListener('visibilitychange', handleVisibilityChange)
  await requestWakeLock()
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
  document.removeEventListener('fullscreenchange', handleFullscreenChange)
  document.removeEventListener('visibilitychange', handleVisibilityChange)
  if (wakeLock) {
    wakeLock.release()
    wakeLock = null
  }
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
  position: relative;
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
  font-size: min(21vw, 50vh); /* Tightly fit 8 characters across the width */
  font-weight: 700;
  line-height: 1;
  text-shadow: 0 0 30px rgba(255, 255, 255, 0.3);
  white-space: nowrap;
  text-align: center;
}

.fullscreen-button {
  position: absolute;
  top: 1.5rem;
  right: 1.5rem;
  background: rgba(255, 255, 255, 0.1);
  border: none;
  border-radius: 50%;
  color: #fff;
  padding: 10px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.3s ease, transform 0.2s ease;
  z-index: 10;
  opacity: 0.5;
}

.fullscreen-button:hover {
  background: rgba(255, 255, 255, 0.2);
  opacity: 1;
  transform: scale(1.1);
}

.fullscreen-button:active {
  transform: scale(0.95);
}
</style>