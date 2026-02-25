<template>
  <div class="app-root">
    <div
      class="app-bg"
      :class="{ loaded: loaded }"
      :style="wallpaper ? { backgroundImage: `url(${wallpaper})` } : {}"
      aria-hidden="true"
    ></div>
    <div class="app-content">
      <Home />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Home from './views/Home.vue'

const wallpaper = ref('')
const loaded = ref(false)

async function pickAndPreload() {
  let list = []
  try {
    const res = await fetch('/wallpapers/manifest.json')
    if (res.ok) list = await res.json()
  } catch (e) {}

  if (!Array.isArray(list) || !list.length) {
    list = [
      '10-10-6k.jpg',
      '10-11-6k.jpg',
      '10-12-6k.jpg',
      '10-13-6k.jpg',
      '10-9-6k.jpg',
      '14-Sonoma-Horizon.png',
      '15-Sequoia-Sunrise.png',
      'big-sur-6016x6016-1476.jpg',
      'big-sur-6016x6016-1492.jpg',
      'big-sur-6016x6016-1496.jpg',
      'big-sur-6016x6016-1497.jpg',
      'big-sur-6016x6016-1498.jpg',
      'big-sur-6016x6016-1499.jpg',
      'coastline-6016x6016-3989.jpg',
      'coastline-6016x6016-3994.jpg',
      'coastline-6016x6016-3995.jpg',
      'macos-big-sur-3200x3200-3774.jpg',
      'macos-big-sur-6016x6016-3776.jpg',
      'macos-big-sur-6016x6016-3777.jpg',
      'macos-big-sur-6016x6016-3778.jpg',
      'macos-big-sur-6016x6016-3781.jpg',
      'macos-big-sur-6016x6016-3783.jpg',
      'macos-big-sur-6016x6016-3784.jpg',
      'macos-big-sur-6016x6016-3785.jpg',
      'macos-big-sur-6016x6016-3795.jpg',
      'macos-big-sur-6016x6016-4004.jpg',
      'macos-big-sur-6016x6016-4011.jpg',
      'macos-catalina-6016x6016-4009.jpg',
      'macos-catalina-6016x6016-4013.jpg',
      'macos-catalina-6016x6016-4015.jpg',
      'macos-catalina-6016x6016-4019.jpg',
      'macos-catalina-6016x6016-4021.jpg',
      'macos-catalina-6016x6016-4022.jpg',
      'macos-mojave-dynamic-wallpaper-5120x2880-32948.jpg',
      'macos-mojave-dynamic-wallpaper-5120x2880-32949.jpg',
      'macos-mojave-dynamic-wallpaper-5120x2880-32950.jpg',
      'macos-mojave-dynamic-wallpaper-5120x2880-32952.jpg',
      'macos-tahoe-26-5120x3413-31202.jpg',
      'macos-tahoe-beach-dawn-6016x3384-32930.jpg',
      'macos-tahoe-beach-day-6016x3384-32926.jpg',
      'macos-tahoe-beach-dusk-6016x3384-32927.jpg',
      'macos-tahoe-beach-night-6016x3384-32928.jpg',
      'road-6016x6016-3996.jpg',
    ]
  }

  const choice = `/wallpapers/${list[Math.floor(Math.random() * list.length)]}`
  const img = new Image()
  img.src = choice
  img.onload = () => {
    wallpaper.value = choice
    requestAnimationFrame(() => (loaded.value = true))
  }
  img.onerror = () => {
    wallpaper.value = choice
    loaded.value = true
  }
}

onMounted(pickAndPreload)
</script>

<style>
html,
body,
#app {
  height: 100%;
}
.app-root {
  min-height: 100vh;
}
.app-bg {
  position: fixed;
  inset: 0;
  z-index: 0;
  background-color: #000;
  background-size: cover;
  background-position: center center;
  background-repeat: no-repeat;
  opacity: 0;
  filter: blur(24px) scale(1.03);
  transition:
    opacity 1.5s ease,
    filter 1.5s ease;
  pointer-events: none;
}
.app-bg::before {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.65);
  transition: opacity 1.5s ease;
}
.app-bg.loaded {
  opacity: 1;
  filter: blur(0px) scale(1);
}
.app-bg.loaded::before {
  opacity: 0;
}
.app-content {
  position: relative;
  z-index: 1;
}
</style>
