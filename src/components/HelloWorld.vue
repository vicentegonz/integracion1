<template>
  <l-map style="height: 450px" :zoom="zoom" :center="center" :maxZoom="9" :minZoom="1">
    <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
    <div v-for="f in Object.keys(fligts.flights)" :key="f">
      <l-marker @click="showPopup(fligts.flights[f].departure.name)" :lat-lng=[fligts.flights[f].departure.location.lat,fligts.flights[f].departure.location.long] >
        <l-icon 
          :icon-size="dynamicSize"
          :icon-anchor="dynamicAnchor"
          icon-url="dep.png" 
          >
        </l-icon>
      </l-marker>
      <l-marker class="dep" @click="showPopup(fligts.flights[f].destination.name)" :lat-lng=[fligts.flights[f].destination.location.lat,fligts.flights[f].destination.location.long] >
      </l-marker>
    </div>
    <l-polyline v-for="f in Object.keys(fligts.flights)" :key="f" :lat-lngs=[[fligts.flights[f].departure.location.lat,fligts.flights[f].departure.location.long],[fligts.flights[f].destination.location.lat,fligts.flights[f].destination.location.long]] ></l-polyline>
    <div v-for="p in Object.keys(planes)" :key="p">
      <l-polyline v-if="Object.keys(fligts.flights).includes(p)" :lat-lngs=planes[p].latlong :color="color"></l-polyline>
      <l-marker v-if="Object.keys(fligts.flights).includes(p)" @click="showPlanePopup(p)" :lat-lng=[planes[p].plane.position.lat,planes[p].plane.position.long] >
        <l-icon v-if="planes[p].plane.status=='arrived'" 
          :icon-size="dynamicSize"
          :icon-anchor="dynamicAnchor"
          icon-url="landed.png" 
          >
        </l-icon>
        <l-icon v-if="planes[p].plane.status=='crashed'" 
          :icon-size="dynamicSize"
          :icon-anchor="dynamicAnchor"
          icon-url="crash.png" 
          >
        </l-icon>
        <l-icon v-if="planes[p].plane.status=='take-off'" 
          :icon-size="dynamicSize"
          :icon-anchor="dynamicAnchor"
          icon-url="takeoff.png" 
          >
        </l-icon>
        <l-icon v-if="planes[p].plane.status=='flying'" 
          :icon-size="dynamicSize"
          :icon-anchor="dynamicAnchor"
          icon-url="plane.png" 
          >
        </l-icon>
      </l-marker>
      <l-marker v-else @click="showPlanePopup(p)" :lat-lng=[planes[p].plane.position.lat,planes[p].plane.position.long] :visible="false">
      </l-marker>
    </div>
  </l-map>
</template>

<script>
import "leaflet/dist/leaflet.css"
import {LMap, LTileLayer, LMarker, LPolyline, LIcon} from '@vue-leaflet/vue-leaflet';

export default {
  props: ['fligts', 'crashed', 'landed', 'takeoff', 'planes'],
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPolyline,
    LIcon,
  },
  data () {
    return {
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
        '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 3,
      center: [51.505, -0.159],
      markerLatLng: [51.504, -0.159],
      color: 'red',
      iconSize: [32, 37],
      iconAnchor: [16, 37]
    };
  },
  methods:{
    showPlanePopup(val) {
      var cap = null;
      var aero = null
      var sig = null
      var eta = null
      var estado = null
      var dist = null
      var hra = null
      var lat = null
      var long = null
      Object.keys(this.planes).forEach(e => {
        if (e == val){
          cap = this.planes[e].plane.captain
          aero = this.planes[e].plane.airline.name
          sig = this.planes[e].plane.airline.id
          eta = this.planes[e].plane.ETA
          estado = this.planes[e].plane.status
          dist = this.planes[e].plane.distance
          hra = this.planes[e].plane.arrival
          lat = this.planes[e].plane.heading.lat
          long = this.planes[e].plane.heading.long

        }
      });
      alert(`Id vuelo: ${val} \n
      Aerolinea: ${aero} (${sig})\n
      Capitan: ${cap}\n
      ETA: ${eta}\n
      Estado del vuelo: ${estado}\n
      Distancia al destino: ${dist}\n
      Hora de llegada estimada: ${hra}\n
      Direccion: ${lat}, ${long}`)
    },
    showPopup(val){
      console.log(val)
      var dep = []
      var dest = []
      var country = null
      var city = null
      Object.keys(this.fligts.flights).forEach(key => {
        if (this.fligts.flights[key].departure.name == val){
          dep.push(key)
          country = this.fligts.flights[key].departure.city.country.name
          city = this.fligts.flights[key].departure.city.name
        }
        if (this.fligts.flights[key].destination.name == val){
          dest.push(key)
          country = this.fligts.flights[key].destination.city.country.name
          city = this.fligts.flights[key].destination.city.name
        }
      });
      alert(`Aeropuerto: ${val} que se encuentra en ${city}, ${country} \n
      Vuelos hacia el aeropuerto: ${dest} \n
      Vuelos desde el aeropuert: ${dep}`)
    }
  },
  computed: {
    dynamicSize () {
      return [this.iconSize, this.iconSize * 1.15];
    },
    dynamicAnchor () {
      return [this.iconSize / 2, this.iconSize * 1.15];
    }
  }
}
</script>

<style>
  .dep {
    background-color: crimson;
  }
</style>

