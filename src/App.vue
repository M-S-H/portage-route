<script setup lang="ts">
import { Loader } from '@googlemaps/js-api-loader';
import { Ref, nextTick, onBeforeMount, onMounted, reactive, ref, watch } from 'vue'
import svg from './assets/canoe.svg'

const loaded = reactive({ map: false, markers: false, places: false, mounted: false, location: false })
const map = ref(null) as Ref<null | google.maps.Map>
const directionsService = ref(null) as Ref<null | google.maps.DirectionsService>
const directionsRenderer = ref(null) as Ref<null | google.maps.DirectionsRenderer>
const startLocation = ref(null) as Ref<null | google.maps.LatLngLiteral>
const mapLib = ref(null) as Ref<null | google.maps.MapsLibrary>
const markerLib = ref(null) as Ref<null | google.maps.marker.AdvancedMarkerElement>
const placesService = ref(null) as Ref<null | google.maps.places.PlacesService>
const endLocation = ref(null) as Ref<null | google.maps.LatLngLiteral>
const directionsResult = ref(null) as Ref<null | google.maps.DirectionsResult>
const mode = ref('selectPoint')



onMounted(() => {
  const loader = new Loader({
    apiKey: import.meta.env.VITE_MAPS_API_KEY,
    version: "weekly"
  })

  loader.importLibrary('maps').then(() => {
    loaded.map = true
    // const mapEl = document.getElementById('map') as HTMLElement
    // const pos = { lng: 39.8283, lat: -98.5795 }

    // if (navigator.geolocation) {
    //   navigator.geolocation.getCurrentPosition((position: GeolocationPosition) => {
    //     pos.lat = position.coords.latitude
    //     pos.lng = position.coords.longitude
    //   })

    //   startLocation.value = pos
    //   console.log(startLocation.value)
    // } else {
    //   return
    // }

    // map.value = new google.maps.Map(mapEl, {
    //   center: { lat: 35.102807127601984, lng: -106.61427345290559 },
    //   zoom: 12,
    //   mapId: 'PM',
    //   disableDefaultUI: true
    // })

    // directionsService.value = new google.maps.DirectionsService()
    // directionsRenderer.value = new google.maps.DirectionsRenderer({
    //   map: map.value
    // })


      // map.value.addListener('click', showDirections)
    })

  loader.importLibrary('marker').then(() => {
    loaded.markers = true
  })

  loader.importLibrary('places').then(() => {
    loaded.places = true
  })
})

onMounted(() => {
  askForLocation()
  loaded.mounted = true
})

watch(loaded, () => {
  if (loaded.map && loaded.markers && loaded.places && loaded.mounted && loaded.location) {
    nextTick(() => initializeMap())
    // setTimeout(() => {
    //   initializeMap()
    //   // nexttick(initializeMap())
    // }, 5000);
  }
})

function askForLocation () {
  const pos = { lat: 0, lng: 0 }
  // if (navigator.geolocation.)
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition((position: GeolocationPosition) => {
      pos.lat = position.coords.latitude
      pos.lng = position.coords.longitude
      console.log('dope')
      loaded.location = true
      startLocation.value = pos
    }, (err: GeolocationPositionError) => {
      console.log(err)
    })
  } else {
    return
  }
}

function initializeMap () {
  const mapEl = document.getElementById('map') as HTMLElement

  map.value = new google.maps.Map(mapEl, {
    center: startLocation.value,
    zoom: 12,
    mapId: 'PM',
    disableDefaultUI: true
  })

  console.log(svg)

  directionsService.value = new google.maps.DirectionsService()
  directionsRenderer.value = new google.maps.DirectionsRenderer({
    map: map.value,
    polylineOptions: {
      strokeColor: 'green', // Set transparent color to hide default path
      strokeOpacity: 1,
      icons: [{
        icon: {
    path: "M 37.5,3.35 12.5,3.35 0,25 12.5,46.65 37.5,46.65 50,25",
    fillColor: "blue",
    fillOpacity: 1,
    strokeWeight: 0,
    rotation: 0,
    scale: 0.3,
    anchor: new google.maps.Point(0, 10),
  },
        offset: '-10',
        repeat: '20px' // Repeat pattern every 20 pixels
      }]
    }
  })

  map.value.addListener('click', showDirections)
}

function showDirections (event: google.maps.MapMouseEvent) {
  // Add Markers
  const startMarker = new google.maps.marker.AdvancedMarkerElement({
    position: startLocation.value,
    map: map.value
  })

  const endingMarker = new google.maps.marker.AdvancedMarkerElement({
    position: event.latLng!,
    map: map.value
  })

  const request: google.maps.DirectionsRequest = {
    origin: startLocation.value!,
    destination: event.latLng!,
    travelMode: google.maps.TravelMode.WALKING
  };

  directionsService.value?.route(request, (result, status) => {
    directionsResult.value = result
    if (status === 'OK') {
      directionsRenderer.value?.setDirections(result);
    } else {
      console.error('Directions request failed due to ' + status);
    }
  });
}
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
  height: 90%;
  background-color: green;
}
</style>
