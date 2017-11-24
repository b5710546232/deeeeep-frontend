<template>
  <div class="hello">
  Home
  <input type="text">
  <div class="map-container">
    <gmap-map
    :center="center"
    :zoom="7"
    style="width: 100%; height: 100%"
  >
    <gmap-marker
      :key="index"
      v-for="(m, index) in markers"
      :position="m.position"
      :clickable="true"
      :draggable="true"
      @click="center=m.position"
    ></gmap-marker>
  </gmap-map>
  </div>
  </div>
</template>

<script>
 import * as VueGoogleMaps from 'vue2-google-maps';
  import Vue from 'vue';
const TOKEN = 'AIzaSyByrrHtlYYgMlya1-fV3qfOfkUrftFtbfU'
const VERSION_NUMBER = '3.24'
  Vue.use(VueGoogleMaps, {
    load: {
      key: TOKEN,
      v: VERSION_NUMBER,
      // libraries: 'places', //// If you need to use place input
    }
  });
export default {
  name: 'Home',
  data () {
      return {
        center: {lat: 13.7563, lng: 100.5018},
        markers: [{
          position: {lat: 13.7563, lng: 100.5018}
        }]
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.map-container{
  height:90vh;
  width:100%;
}
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
