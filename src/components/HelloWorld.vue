<template>
  <l-map style="height: 450px" :zoom="zoom" :center="center" :maxZoom="9" :minZoom="1">
    <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
    <l-marker :lat-lng="markerLatLng"></l-marker>
    <l-polyline v-for="f in Object.keys(fligts.flights)" :key="f" :lat-lngs=[[fligts.flights[f].departure.location.lat,fligts.flights[f].departure.location.long],[fligts.flights[f].destination.location.lat,fligts.flights[f].destination.location.long]] :color="color"></l-polyline>
  </l-map>
</template>

<script>
import "leaflet/dist/leaflet.css"
import {LMap, LTileLayer, LMarker, LPolyline} from '@vue-leaflet/vue-leaflet';

export default {
  props: ['fligts'],
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPolyline,
  },
  data () {
    return {
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
        '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 3,
      center: [51.505, -0.159],
      markerLatLng: [51.504, -0.159],
      color: 'green'
    };
  }
}
</script>

