<script setup lang="ts">
import { Loader } from '@googlemaps/js-api-loader';
import { Ref, nextTick, onMounted, ref } from 'vue'

const map = ref(null) as Ref<null | google.maps.Map>
const startLocation = ref(null) as Ref<null | string>
const endLocation = ref(null) as Ref<null | string>


onMounted(() => {
  const loader = new Loader({
    apiKey: import.meta.env.VITE_MAPS_API_KEY,
    version: "weekly"
  })

  const mapEl = document.getElementById('map') as HTMLElement

  const pos = { lng: 39.8283, lat: 98.5795 }

  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition((position: GeolocationPosition) => {
      pos.lat = position.coords.latitude
      pos.lng = position.coords.longitude
    })
  }

  loader
    .importLibrary('maps')
    .then(({Map}) => {
      console.log('here I am')
      map.value = new Map(mapEl, {
        center: pos,
        zoom: 12
      })
      console.log(map.value)
    })
    .catch((e) => {
      console.log(e)
  })
})
</script>

<template>
  <div class="portage-route">
    <div id="map">
    </div>
  </div>
</template>

<style scoped>
.portage-route {
  width: 100%;
  height: 100%;
}

#map {
  width: 100%;
  height: 100%;
  background-color: green;
}
</style>
