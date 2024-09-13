<script setup lang="ts">
import { Loader } from '@googlemaps/js-api-loader';
import { Ref, nextTick, onMounted, reactive, ref, watch } from 'vue'
import canoePath from './assets/canoe.txt?raw'
import dq from './assets/dq.png'
import start from './assets/start.png'
import end from './assets/end.png'

const loaded = reactive({ map: false, markers: false, places: false, mounted: false, location: false })
const directionsService = ref(null) as Ref<null | google.maps.DirectionsService>
const directionsRenderer = ref(null) as Ref<null | google.maps.DirectionsRenderer>
const startLocation = ref(null) as Ref<null | google.maps.LatLngLiteral>
const placesService = ref(null) as Ref<null | google.maps.places.PlacesService>
const endLocation = ref(null) as Ref<null | google.maps.LatLngLiteral>
const directionsResult = ref(null) as Ref<null | google.maps.DirectionsResult>
const distance = ref(0)
const goToDq = ref(true)

const icons = reactive({
  start: null,
  end: null,
  dq: null
} as { [key: string]: null | google.maps.marker.AdvancedMarkerElement})

let map = null as any


onMounted(() => {
  const loader = new Loader({
    apiKey: import.meta.env.VITE_MAPS_API_KEY,
    version: 'weekly'
  })

  loader.importLibrary('maps').then(() => {
    loaded.map = true
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
    // initTest()
    nextTick(() => initializeMap())
  }
})

function askForLocation () {
  const pos = { lat: 0, lng: 0 }
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition((position: GeolocationPosition) => {
      pos.lat = position.coords.latitude
      pos.lng = position.coords.longitude
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
  askForLocation()
  const mapEl = document.getElementById('map') as HTMLElement

  map = new google.maps.Map(mapEl, {
    center: startLocation.value,
    zoom: 12,
    mapId: '4504f8b37365c3d0'
  })

  directionsService.value = new google.maps.DirectionsService()
  directionsRenderer.value = new google.maps.DirectionsRenderer({
    map: map,
    suppressMarkers: true,
    polylineOptions: {
      strokeColor: '#008bff', // Set transparent color to hide default path
      strokeOpacity: 0.5,
      icons: [{
        icon: {
          path: canoePath,
          fillColor: "#008bff",
          fillOpacity: 1,
          strokeWeight: 0,
          rotation: 90,
          scale: 1,
          anchor: new google.maps.Point(5, 0),
        },
        repeat: '28px'
      }]
    }
  });

  const startIcon = document.createElement('img')
  startIcon.src = start
  startIcon.width = 40
  new google.maps.marker.AdvancedMarkerElement({
    map: map,
    position: startLocation.value,
    content: startIcon,
  })

  placesService.value = new google.maps.places.PlacesService(map)

  map.addListener('click', handleLocationSelection)
}


function handleLocationSelection(event: google.maps.MapMouseEvent) {
  console.log('yoo')
  endLocation.value = { lat: event.latLng?.lat()!, lng: event.latLng?.lng()!}
  showDirections()
}

function clearDirections () {
  endLocation.value = null
  // directionsResult.value
  // directionsRenderer.value?.setMap(null)
  
  if (icons.dq) {
    icons.dq.map = null
  }

  if (icons.end) {
    icons.end.map = null
  }
  // directionsRenderer.value?.setDirections(null)
}

function setDirections (waypoints: any[]) {
  console.log('set directions')
  const request: google.maps.DirectionsRequest = {
    origin: startLocation.value!,
    destination: endLocation.value!,
    travelMode: google.maps.TravelMode.WALKING,
    waypoints: waypoints,
  };

  directionsService.value?.route(request, (result, status) => {
    directionsResult.value = result

    let meters = 0
    result?.routes[0].legs.forEach(leg => {
      meters += leg.distance?.value!
    })

    distance.value = meters / 5.0292
    
    if (status === 'OK') {
      directionsRenderer.value?.setDirections(result);
    } else {
      console.error('Directions request failed due to ' + status);
    }
  });

  const endIcon = document.createElement('img')
  endIcon.src = end
  endIcon.width = 40
  icons.end = new google.maps.marker.AdvancedMarkerElement({
    map: map,
    position: endLocation.value!,
    content: endIcon
  })
}


function showDirections () {
  // new google.maps.marker.AdvancedMarkerElement({
  //   position: endLocation.value,
  //   map: map.value
  // })

  if (!startLocation.value || !endLocation.value) {
    return
  }

  if (icons.dq) {
    icons.dq.map = null
  }

  if (icons.end) {
    icons.end.map = null
  }


  // Find Dairy Queen
  const dqRequest = {
    location: endLocation.value!,
    radius: 50000,
    keyword: 'Dairy Queen'
  }

  const centerRequest = {
    location: {
      lat: (endLocation.value.lat + startLocation.value.lat) / 2,
      lng: (endLocation.value.lng + startLocation.value.lng) / 2
    },
    radius: 50000,
    keyword: 'Dairy Queen'
  }

  const waypoints = [] as google.maps.DirectionsWaypoint[]
  if (goToDq.value) {
    placesService.value?.nearbySearch(centerRequest, (results, status) => {
      let nearestDQ = null
      
      if (status === google.maps.places.PlacesServiceStatus.OK && results && results.length > 0) {
        console.log(results)
        setDqDirections(results[0])
      } else {
        placesService.value?.nearbySearch(dqRequest, (results, status) => {
          let nearestDQ = null
          
          if (status === google.maps.places.PlacesServiceStatus.OK && results && results.length > 0) {
            setDqDirections(results[0])
          } else {
            setDirections([])
          }
        })
      }
    })
  } else {
    console.log('here')
    setDirections([])
  }
}

function setDqDirections (nearestDQ: any) {
  const img = document.createElement('img')
  img.src = dq
  img.width = 40
  icons.dq = new google.maps.marker.AdvancedMarkerElement({
    map: map,
    position: nearestDQ!.geometry!.location,
    content: img
  });

  const waypoints = []

  waypoints.push({
    location: nearestDQ.geometry?.location
  })

  setDirections(waypoints)
}
</script>

<template>
  <div class="portage-route">
    <div id="map">
    </div>
    <div class="no-selected" v-if="endLocation">
      Selected
      <button @click="clearDirections">Clear</button>
    </div>
    <div class="selected" v-if="!endLocation">
      Select a place on the map to portage to.
    </div>
  </div>
</template>

<style scoped>
.portage-route {
  width: 100%;
  height: 100%;

  font-family: sans-serif;
}

#map {
  width: 100%;
  height: calc(100% - 60px);
  background-color: green;
}

.not-selected {
  text-align: center;
  line-height: 60px;
}
</style>
