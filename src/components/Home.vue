<template>
  <div class="home">
    <div class="input-search-container">
      <div class="search-bar">
      <gmap-autocomplete  :value="description" class="form-control" @place_changed="setPlace">
    </gmap-autocomplete>
    </div>
    </div>
    <div class="map-container container">
      <gmap-map :center="latLng" :zoom="zoom" style="width: 100%; height: 100%">
        <gmap-marker :key="index" v-for="(m, index) in markers" :position="m.position" @position_changed="updateChild(m, 'position', $event)"
          :clickable="true" :draggable="true" @click="onClickMarker(m)" @dragend="onDragEnd(m)"></gmap-marker>
      </gmap-map>
    </div>

    <!--<div class="">marker : lat{{this.markers[0].position.lat.toFixed(6)}}, lng: {{this.markers[0].position.lng.toFixed(6)}}</div>-->
    <!--<div class="">{{latLng.lat}}, {{latLng.lng}}</div>-->
    <modal name="hello-world" class="information-modal" :scrollable="true" :adaptive="true" width="80%" height="auto">
      <div class="text-center">
        <!--<img class="img" src="../assets/mock.png" alt="">-->
        <div class="container">
          <full-calendar :events="calendarEvents" :firstDay="3" locale="en"></full-calendar>
        </div>
      </div>
    </modal>
  </div>
</template>

<script>
  import moment from 'moment';
  import * as VueGoogleMaps from 'vue2-google-maps';
  import Vue from 'vue';
  import axios from 'axios'
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
    data() {
      return {
        zoom: 9,
        description: 'ประเทศไทย',
        latLng: { lat: 13.7563, lng: 100.5018 },
        markers: [{
          position: { lat: 13.7563, lng: 100.5018 }
        }],
        calendarEvents: [
          {
            // Home.vue?6e55:95 max 2017-10-30 23:00:00 min 2017-09-31 09:00:00
            title : 'rain',
            start : moment(),
            end : moment().add(3,'days')
          }
        ],
      }
    },
    mounted() {
      this.fetchRain()
    },
    methods: {
      setDescription(description) {
        this.description = description;
      },
      updateChild(object, field, event) {
        if (field === 'position') {
          object.position = {
            lat: event.lat(),
            lng: event.lng(),
          };
        }
      },
      onClickMarker(marker) {
        this.$nextTick(() => {
          this.$modal.show('hello-world')
          // latLng=m.position
        })
        // alert('ok')
      },
      fetchRain(){
        const lat_suffix = '.362634'
        const lng_suffix = '.986175'
         let lat = this.markers[0].position.lat.toFixed(0)+lat_suffix
        let lng = this.markers[0].position.lng.toFixed(0)+lng_suffix
        let API = `http://54.169.99.243:1921/rain/${lat}/${lng}`
        return axios.get(API).then((res)=>{
          let data = JSON.parse(res.data)
          let max = moment(data[0].date)
          let min = moment(data[0].date)
          
          let maxDate = data.sort((a,b)=>{ return ( moment(a.date) - moment(b.date) ); }).pop().date;
          let minDate = data.sort((b,a)=>{ return ( moment(a.date) - moment(b.date) ); }).pop().date;
          this.calendarEvents = [
             {
              title : 'rain',
              start :moment(minDate),
              end : moment(maxDate)
            }
          ]
          console.warn('max',maxDate,'min',minDate)
          return data
        }).catch((error)=>{
          console.error(error)
        })
      },
      onDragEnd(marker) {
        console.log('drag', marker.position)
        this.fetchRain()
        //  this.latLng = {
        //   lat: marker.position.lat,
        //   lng: marker.position.lng
        // }
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
  .search-bar{
    padding-left:1%;
    padding-right:1%;
    padding-bottom:1%;
  }
  .input-search-container{
    background:#2C6FD5;
    margin-bottom:10px;
  }
  .comp-full-calendar{
    margin-bottom:10%;
  }
  
  .information-modal {
  }
  
  .infomation-modal-content {
    height: 100%;
    box-sizing: border-box;
    padding: 10px;
    font-size: 13px;
    overflow: auto;
  }
  
  .map-container {
    height: 70vh;
    width: 100%;
  }
  
  h1,
  h2 {
    font-weight: normal;
  }
  
  .img {
    max-width: 80%;
    max-height: 80%;
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