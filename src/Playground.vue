<script setup lang="ts">
import ElevateZoom from "../lib/components/ElevateZoom/ElevateZoom.vue";
</script>

<template>
  <div class="playground">
    <h1>Playground</h1>
    <ElevateZoom
      :image="'https://picsum.photos/id/237/200/300'"
      :origin-image="'https://picsum.photos/id/237/400/600'"
      :is-active-elevate-zoom="false"
    />
  </div>
</template>

<style lang="css" scoped>
* {
  margin: 0;
  padding: 0;
}
.playground {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  padding: 1rem;
}
.playground > h1 {
  padding: 1rem 0;
  border-bottom: 0.1rem dashed #ccc;
}
</style>
