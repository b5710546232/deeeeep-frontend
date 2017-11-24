<template>
  <div class="home">
  Home
  

       <gmap-autocomplete :value="description"
       class="form-control"
        @place_changed="setPlace">
      </gmap-autocomplete>
       {{latLng.lat}},
    {{latLng.lng}}
  <div class="map-container container">
    <gmap-map
    :center="latLng"
    :zoom="zoom"
    style="width: 100%; height: 100%"
  >
    <gmap-marker
      :key="index"
      v-for="(m, index) in markers"
      :position="m.position"
      :clickable="true"
      :draggable="true"
      @click="onClickMarker(m)"
    ></gmap-marker>
  </gmap-map>
  </div>

<modal name="hello-world">
  hello, world!
</modal>

  </div>
</template>

<script>
 import * as VueGoogleMaps from 'vue2-google-maps';
  import Vue from 'vue';
const TOKEN = 'AIzaSyByrrHtlYYgMlya1-fV3qfOfkUrftFtbfU'
  Vue.use(VueGoogleMaps, {
    load: {
      key: TOKEN,
      libraries: 'places'
      // libraries: 'places', //// If you need to use place input
    }
  });
export default {
  name: 'Home',
  data () {
      return {
        zoom:9,
        description: 'ประเทศไทย',
        latLng: {lat: 13.7563, lng: 100.5018},
        markers: [{
          position: {lat: 13.7563, lng: 100.5018}
        }]
      }
  },
  mounted(){

  },
  methods: {
      setDescription(description) {
        this.description = description;
      },
      onClickMarker(marker){
            this.$nextTick(() => {
        this.$modal.show('hello-world')
      })
        // alert('ok')
      },
      setPlace(place) {
        this.latLng = {
          lat: place.geometry.location.lat(),
          lng: place.geometry.location.lng(),
        }
        this.zoom = 10
        this.markers[0].position = this.latLng
      }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.infomation-modal-content {
  height: 100%;
  box-sizing: border-box;
  padding: 10px;
  font-size: 13px;
  overflow: auto;
}

.map-container{
  height:70vh;
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
