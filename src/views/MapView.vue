<template>
  <div id="app">
    <div ref="mapContainer" class="map"></div>
    <div class="overlay-text">2d map</div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import L from "leaflet";
import "leaflet/dist/leaflet.css";
import "leaflet.polyline.snakeanim/L.Polyline.SnakeAnim";

const mapContainer = ref(null);

onMounted(() => {
  const map = L.map(mapContainer.value).setView([22.3375, 114.1747], 11);

  //base map
  L.tileLayer(
    "https://mapapi.geodata.gov.hk/gs/api/v1.0.0/xyz/basemap/wgs84/{z}/{x}/{y}.png",
    {
      attribution: "© GeoData.gov.hk",
    }
  ).addTo(map);

  //on top adding map label
  L.tileLayer(
    "https://mapapi.geodata.gov.hk/gs/api/v1.0.0/xyz/label/hk/tc/wgs84/{z}/{x}/{y}.png",
    {
      attribution: "",
      transparent: true,
    }
  ).addTo(map);

  // Sample MTR lines (simplified for demo)
  const mtrRoutes = [
    {
      name: "Tsuen Wan Line",
      color: "red",
      coords: [
        // [22.3739, 114.1194], // Tsuen Wan
        // [22.3705, 114.1365], // Kwai Hing
        [22.3375, 114.1747], // Prince Edward
        [22.3185, 114.1703], // Tsim Sha Tsui
        [22.3045, 114.185],  // Admiralty
      ],
    },
   
  ]

  // Draw each route
  mtrRoutes.forEach((route) => {
    const poly = L.polyline(route.coords, {
      color: route.color,
      weight: 4,
    }).addTo(map);

    // Add station markers
    route.coords.forEach((coord, i) => {
      L.circleMarker(coord, {
        radius: 5,
        color: "#000",
        weight: 1,
        fillColor: route.color,
        fillOpacity: 0.9,
      })
        .addTo(map)
        .bindPopup(`${route.name} - Station ${i + 1}`);
    });

    // Animate
    if (poly.snakeIn) {
      setTimeout(() => poly.snakeIn(), 600); // delay for smoother effect
    }
  });

  const imageUrl = "/a-tall-building-background.png";



  const imageBounds = [
  [22.3040, 114.1845], // SW
  [22.3080, 114.1865], // NE
];
  const overlay = L.imageOverlay(imageUrl, imageBounds).addTo(map);

});
</script>

<style>
html,
body,
#app {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}

#app {
  display: flex;
  flex-direction: column;
}

.map {
  flex: 1;
  width: 100%;
}

.overlay-text {
  position: absolute;
  top: 10px;
  left: 10px;
  font-size: "24px";
  color: rgb(210, 49, 49);
  z-index: 1000;
}
</style>
