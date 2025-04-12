<template>
  <div class="w-full max-w-md p-7 space-y-4 bg-black text-white">
    <h2 class="text-lg font-bold mb-4">Locations</h2>
    <button
      v-for="loc in locations"
      :key="loc.name"
      :disabled="isMoving"
      @click="$emit('select-location', loc)"
      class="block w-full text-left px-4 py-2 bg-gray-700 hover:bg-gray-600 rounded disabled:opacity-50 disabled:cursor-not-allowed"
    >
      {{ loc.name }}
    </button>
  </div>
</template>

<script setup>
import rawLocations from "../../data/locationList.json";

defineProps({
  isMoving: Boolean
});

const locations = rawLocations;
</script>