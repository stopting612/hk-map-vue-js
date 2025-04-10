<template>
  <div class="flex h-screen w-screen">
    <!-- Sidebar on the left -->
    <LocationNavbar @select-location="moveToLocation" />

    <!-- Map on the right -->
    <div ref="mapContainer" class="flex-1 relative">
      <div class="overlay-text">2d map</div>
    </div>
  </div>

 

</template>

<script setup>
import { onMounted, ref } from "vue";
import LocationNavbar from '../components/LocationNavbar.vue'

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
        [22.3045, 114.185], // Admiralty
      ],
    },
  ];

  const motionPolyline = L.motion
    .polyline(
      [
        [22.3027, 114.1772], // Central
        [22.2783, 114.1747], // Causeway Bay
        [22.3364, 114.1745], // Kowloon Tong
      ],
      { color: "blue" },
      {
        auto: true,
        duration: 9800,
        easing: L.Motion.Ease.easeInOutQuart,
      },
      {
          removeOnEnd: false,
          showMarker: true,
          icon: L.divIcon({
    html: "<img src='/car.svg' style='width: 30px; height: 30px;' />",
    iconSize: L.point(30, 30),
    className: ''
  })
      }
    )
    .addTo(map);

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
    [22.304, 114.1845], // SW
    [22.308, 114.1865], // NE
  ];
  const overlay = L.imageOverlay(imageUrl, imageBounds).addTo(map);
});
</script>

<style>

</style>
