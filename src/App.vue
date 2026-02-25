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
const CACHE_NAME = 'wallpapers-cache-v1'
let currentObjectUrl = null

async function fetchAndCacheManifest() {
  const cache = await caches.open(CACHE_NAME)
  const manifestUrl = '/wallpapers/manifest.json'
  let res = await cache.match(manifestUrl)
  if (!res) {
    try {
      res = await fetch(manifestUrl)
      if (res && res.ok) await cache.put(manifestUrl, res.clone())
    } catch (e) {
      // ignore network errors
    }
  }
  if (res) return res.json()
  return []
}

async function getCachedOrNetworkImage(path) {
  const cache = await caches.open(CACHE_NAME)
  const match = await cache.match(path)
  if (match && match.ok) {
    const blob = await match.blob()
    return URL.createObjectURL(blob)
  }

  try {
    const net = await fetch(path)
    if (net && net.ok) {
      await cache.put(path, net.clone())
      const blob = await net.blob()
      return URL.createObjectURL(blob)
    }
  } catch (e) {
    // network failed
  }
  return null
}

async function cacheImage(path) {
  const cache = await caches.open(CACHE_NAME)
  const match = await cache.match(path)
  if (match) return
  try {
    const res = await fetch(path)
    if (res && res.ok) await cache.put(path, res.clone())
  } catch (e) {
    // ignore
  }
}

function setWallpaperObjectUrl(objUrl) {
  if (currentObjectUrl) URL.revokeObjectURL(currentObjectUrl)
  currentObjectUrl = objUrl
  wallpaper.value = objUrl
  requestAnimationFrame(() => (loaded.value = true))
}

async function pickAndPreload() {
  const list = await fetchAndCacheManifest()
  if (!list?.length) {
    return
  }

  const existingRaw = localStorage.getItem('wallpaper-name')
  const existingName = existingRaw ? existingRaw.replace(/^\/wallpapers\//, '') : null

  let currentName = null

  // 1) load cached blob for existing wallpaper if present
  if (existingName) {
    console.log(`Loading existing wallpaper from cache: ${existingName}`)
    const existingPath = `/wallpapers/${existingName}`
    const cache = await caches.open(CACHE_NAME)
    const match = await cache.match(existingPath)
    if (match?.ok) {
      const blob = await match.blob()
      const obj = URL.createObjectURL(blob)
      setWallpaperObjectUrl(obj)
      currentName = existingName
    }
  }

  // 2) if no cached existing, pick and load a new image and set as current
  if (!currentName) {
    const choiceName = list[Math.floor(Math.random() * list.length)]
    const choicePath = `/wallpapers/${choiceName}`
    const objUrl = await getCachedOrNetworkImage(choicePath)
    if (objUrl) {
      setWallpaperObjectUrl(objUrl)
      localStorage.setItem('wallpaper-name', choiceName)
      currentName = choiceName
    }
  }

  // 4) prefetch an on-deck wallpaper into cache (don't display)
  const onDeckPool = list.filter((n) => n !== currentName)
  if (onDeckPool.length) {
    const onDeckName = onDeckPool[Math.floor(Math.random() * onDeckPool.length)]
    const onDeckPath = `/wallpapers/${onDeckName}`
    console.log(`Prefetching on-deck wallpaper into cache: ${onDeckName}`)
    localStorage.setItem('wallpaper-name', onDeckName)
    await cacheImage(onDeckPath)
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
