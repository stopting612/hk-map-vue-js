<template>
  <div class="w-500 p-7 space-y-4 bg-black text-white">
    <h2 class="text-lg font-bold mb-4">Locations</h2>
    <button
      v-for="location in locations"
      :key="location.id"
      @click="$emit('select-location', location)"
      class="block w-full text-left px-4 py-2 bg-gray-700 hover:bg-gray-600 rounded"
    >
      {{ location.name }}
    </button>
  </div>
</template>

<script setup>
import rawLocations from '../../data/locationList.json'

const locations = rawLocations
</script>