<template>
  <div>
    <div class="tile-array">
      <div class="tile-cell" v-for="app in apps" :key="app.name">
        <Tile :app="app" />
      </div>
    </div>
  </div>
</template>

<script setup>
import apps from '../data/apps'
import Tile from './Tile.vue'
</script>

<style scoped>
.tile-array {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  padding: 2rem;
}
.tile-cell {
  display: flex;
}
</style>
