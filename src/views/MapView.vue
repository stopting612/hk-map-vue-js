<template>
  <div class="flex h-screen">
    <!-- Sidebar -->
    <LocationNavbar
      :is-moving="isMoving"
      @select-location="moveToLocation"
      class="w-64 bg-gray-800 text-white"
    />

    <!-- Map + UI Container -->
    <div class="flex-1 relative bg-gray-200">
      <div ref="mapContainer" class="absolute inset-0 z-0"></div>

      <button
        class="absolute top-4 left-16 z-50 bg-blue-500 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded shadow"
        @click="togglePopup"
      >
        Button
      </button>

      <div
        v-if="activePopup"
        class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 z-50"
      >
        <div
          class="bg-black text-white p-6 rounded-lg shadow-xl max-w-md w-full border border-gray-200"
        >
          <h2 class="text-xl font-bold mb-3">Confirmation</h2>
          <p class="mb-4">
            Hi I am a popup! This is a simple confirmation message.
          </p>
          <div class="flex space-x-3">
            <button
              class="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded"
              @click="handleAction"
            >
              Action
            </button>
            <button
              class="flex-1 bg-gray-200 hover:bg-gray-300 py-2 px-4 rounded text-black"
              @click="togglePopup"
            >
              Close
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import LocationNavbar from "../components/LocationNavbar.vue";

const mapContainer = ref(null);
const currentPosition = ref([22.3027, 114.1772]); // Starting point
const isMoving = ref(false);

let map = null;
let carMarker = null;
let lastMotionPolyline = null;

const activePopup = ref(false);
const togglePopup = () => {
  activePopup.value = !activePopup.value;
};

onMounted(() => {
  map = L.map(mapContainer.value, {
    zoomControl: false,
  }).setView(currentPosition.value, 13);

  // Disable all map zoom controls
  map.scrollWheelZoom.disable();
  map.doubleClickZoom.disable();
  map.touchZoom.disable();
  map.boxZoom.disable();
  map.keyboard.disable();

  // Base map
  L.tileLayer(
    "https://mapapi.geodata.gov.hk/gs/api/v1.0.0/xyz/basemap/wgs84/{z}/{x}/{y}.png",
    {
      attribution: "© GeoData.gov.hk",
    }
  ).addTo(map);

  // Map label
  L.tileLayer(
    "https://mapapi.geodata.gov.hk/gs/api/v1.0.0/xyz/label/hk/tc/wgs84/{z}/{x}/{y}.png",
    {
      attribution: "",
      transparent: true,
    }
  ).addTo(map);

  // Add a marker at the starting point
  const buildingIcon = L.icon({
    iconUrl: "/a-tall-building-background.png",
    iconSize: [100, 100], // width, height in pixels
    iconAnchor: [20, 40], // anchor point (bottom center)
  });

  L.marker([22.2783, 114.1747]).addTo(map).bindPopup("📍 Central, Hong Kong");

  L.marker([22.3375, 114.1755], { icon: buildingIcon })
    .addTo(map)
    .bindPopup("🏢 Custom Building");

  carMarker = L.marker(currentPosition.value, {
    icon: L.divIcon({
      html: "<img src='/car.svg' style='width: 30px; height: 30px;' />",
      iconSize: L.point(30, 30),
      iconAnchor: [15, 15], // ✅ Center the icon
      className: "",
    }),
  }).addTo(map);
});

function moveToLocation(location) {
  if (isMoving.value) return; // Prevent clicking while moving

  isMoving.value = true;

  const destination = location.coordinates;

  //If already at destination, skip animation
  const start = currentPosition.value;

  if (start[0] === destination[0] && start[1] === destination[1]) {
    isMoving.value = false;
    return;
  }

  // Draw a polyline from current to destination
  L.polyline([start, destination], {
    color: "blue",
    weight: 4,
  }).addTo(map);

  const duration = 6000;
  const steps = 50;
  let step = 0;

  const latDiff = (destination[0] - start[0]) / steps;
  const lngDiff = (destination[1] - start[1]) / steps;

  const interval = setInterval(() => {
    step++;
    const nextLat = start[0] + latDiff * step;
    const nextLng = start[1] + lngDiff * step;
    carMarker.setLatLng([nextLat, nextLng]); // Update car position
    map.panTo([nextLat, nextLng]); // this will make the map follow the car

    if (step >= steps) {
      clearInterval(interval);
      currentPosition.value = destination;
      isMoving.value = false; // ✅ Re-enable the buttons
    }
  }, duration / steps);
}
</script>

<style>
html,
body,
#app {
  margin: 0;
  padding: 0;
  height: 100%;
}

.leaflet-control-attribution {
  display: none !important;
} /*Hide the attribution control*/
</style>
