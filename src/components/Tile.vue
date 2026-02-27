<template>
  <LiquidGlass class="tile-container hover-grow cursor-pointer" :style="{ '--scale': 1.02 }">
    <div class="tile" @click="openLink(app.link)">
      <img
        v-if="app?.icon"
        class="tile-icon m-1 mb-2"
        :src="`/icons/${app.icon}`"
        :alt="app.name || 'icon'"
      />
      <div class="tile-label">{{ app.name }}</div>
    </div>
  </LiquidGlass>
</template>

<script setup>
import LiquidGlass from './LiquidGlass.vue'
const { app } = defineProps({
  app: {
    type: Object,
    required: true,
  },
})

const openLink = (url) => {
  window.open(url, '_self')
}
</script>

<style scoped>
.tile {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  min-height: 200px;
  justify-content: center;
}
.tile-icon {
  width: 10em;
  height: 10em;
  object-fit: contain;
  display: block;
}
.tile-label {
  font-size: 0.85rem;
  text-align: center;
  text-shadow: -1px 1px 3px #00000082;
}
.tile-container {
  padding: 0.5rem;
  width: 100%;
}
.hover-grow {
  transition: all 0.15s ease-in-out;
  transform-origin: 50% 50%;
  transform-box: fill-box;
  -webkit-font-smoothing: subpixel-antialiased;
  will-change: font-size;
}
.hover-grow:hover {
  transform: scale(var(--scale));
  filter: drop-shadow(2px 2px 2px #3f454e00);
}
</style>
