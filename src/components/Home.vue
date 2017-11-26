<template>
  <div class="home">
    <div class="search-container">
      <div class="container">

        <div class="row">
          <div class="col-10">

            <div class="search-bar ">
              <div class="inner-addon left-addon">
                <i class="fa fa-search"></i>
                <gmap-autocomplete :value="description" class="form-control search-input" @place_changed="setPlace">
                </gmap-autocomplete>
              </div>
            </div>
          </div>

          <div class="col-2">
            <button class="button btn app-btn" @click="onClickPredict()">Predict</button>
          </div>

        </div>
      </div>
    </div>
    <div class="map-container">
      <gmap-map :center="latLng" :zoom="zoom" style="width: 100%; height: 100%">
        <gmap-marker :key="index" v-for="(m, index) in markers" :position="m.position" @position_changed="updateChild(m, 'position', $event)"
          :clickable="true" :draggable="true" @click="onClickMarker(m)" @dragend="onDragEnd(m)"></gmap-marker>
      </gmap-map>
    </div>

    <!--<div class="">marker : lat{{this.markers[0].position.lat.toFixed(6)}}, lng: {{this.markers[0].position.lng.toFixed(6)}}</div>-->
    <!--<div class="">{{latLng.lat}}, {{latLng.lng}}</div>-->



    <!--calendar-->
    <modal name="calendar-modal" class="information-modal" :scrollable="true" :adaptive="true" width="80%" height="auto">
      <div class="text-center">
        <div class="container infomation-modal-content">
          <full-calendar :events="calendarEvents" :firstDay="3" locale="en"></full-calendar>
        </div>
      </div>
    </modal>

    <!--predict-->
    <!--มีใส่โล ใส่วัน กด แล้วก้ออกมาเปนปริมาณน้ำฝน-->
    <modal name="predict-modal" class="information-modal" :scrollable="true" :adaptive="true" width="80%" height="300px">
      <div class="text-center">
        <div class="container predict-modal-content">
          <h1>Predict</h1>
          <div class="row">
            <span>Time: </span>
            <date-picker :date="startTime" :option="option" :limit="limit" ></date-picker>
            <div class="btn-submit">
              <button class="btn" @click="onSubmitPredict()">submit</button>
            </div>
          </div>
          <div v-if="this.result.length>0" class="">
            Result Water level : {{result}} unit
          </div>
          <div v-if="this.result.length<=0" class="">
            Select time and submit!
          </div>
        </div>
      </div>
    </modal>



  </div>
</template>

<script>
  import myDatepicker from 'vue-datepicker/vue-datepicker-es6.vue'
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
    components:{
      'date-picker': myDatepicker
    },
    data() {
      return {
        result:'',
        option: {
        type: 'min',
        week: ['Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa', 'Su'],
        month: ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
        format: 'YYYY-MM-DD HH:mm',
        placeholder: 'when?',
        inputStyle: {
          'display': 'inline-block',
          // 'padding': '6px',
          'padding-left' : '5px',
          'line-height': '22px',
          'font-size': '16px',
          'border':'1px solid #44475a',
          'color': '#5F5F5F'
        },
         color: {
          header: '#282a36',
          headerText: '#8be9fd'
        },
        buttons: {
          ok: 'Ok',
          cancel: 'Cancel'
        },
        
        overlayOpacity: 0.5, // 0.5 as default
        dismissible: true // as true as default
      },


    limit: [{
        type: 'weekday',
        available: [1, 2, 3, 4, 5]
      }],

        startTime: {
          time: ''
        },
        zoom: 9,
        description: 'ประเทศไทย',
        latLng: { lat: 13.7563, lng: 100.5018 },
        markers: [{
          position: { lat: 13.7563, lng: 100.5018 }
        }],
        calendarEvents: [
          {
            // Home.vue?6e55:95 max 2017-10-30 23:00:00 min 2017-09-31 09:00:00
            title: 'rain',
            start: moment(),
            end: moment().add(3, 'days')
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
          this.$modal.show('calendar-modal')
          // latLng=m.position
        })
        // alert('ok')
      },
      onSubmitPredict(){
        let date = moment(this.startTime.time).format('YYYY-MM-DD')
        let hour = moment(this.startTime.time).get('hour')
        const lat_suffix = '.362634'
        const lng_suffix = '.986175'
        let lat = this.markers[0].position.lat.toFixed(0) + lat_suffix
        let lng = this.markers[0].position.lng.toFixed(0) + lng_suffix
        console.warn(date,hour)
        this.result = ''
        let API = `http://128.199.88.139:5050/predict/?lat=${lat}&long=${lng}&date=${date}&hour=${hour}`
        axios.get(API).then( (response)=> {
          console.log('predict',response)
          this.result = response.data[0].predict
        }).catch(error=>{
          console.error(error)
        })
      },
      fetchRain() {
        const lat_suffix = '.362634'
        const lng_suffix = '.986175'
        let lat = this.markers[0].position.lat.toFixed(0) + lat_suffix
        let lng = this.markers[0].position.lng.toFixed(0) + lng_suffix
        let API = `http://128.199.88.139:5050/?lat=${lat}&long=${lng}`
        // let API = `http://54.169.99.243:1921/rain/${lat}/${lng}`
        return axios.get(API).then((res) => {
          let data = res.data.map((o) => {
            o['title'] = o['start'] + ': ' + 'rain'
            return o
          })
          data = data.sort(function (a, b) {
            return moment(a.start).isBefore(moment(b.start));
          });
          console.log(data)
          this.calendarEvents = data
          console.log('done')
          return data
        }).catch((error) => {
          console.error(error)
        })
      },
      onClickPredict() {
        this.result = ''
        this.$modal.show('predict-modal')
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
  .btn-submit,  .btn-submit:active,.btn-submit:hover{
    margin-left:50%;
    line-height: 22px;
    font-size: 16px;
    border:1px solid #44475a!important;
    color: #5F5F5F;
    
  }
  .predict-modal-content{
    padding:1%;
    padding-left:100px;
  }

  .search-input {
    padding-left: 2em;
  }
  
  * {
    border-radius: 0px;
  }
  
  .fa-search {
    color: #8be9fd;
  }
  
  .inner-addon {
    position: relative;
  }
  /* style icon */
  
  .inner-addon .fa {
    position: absolute;
    padding: 10px;
    pointer-events: none;
  }
  /* align icon */

  .left-addon .fa {
    left: 0px;
  }
  
  .right-addon .fa {
    right: 0px;
  }
  
  .input-group-addon {
    background: #282a36;
    color: #bd93f9;
    border-color: transparent;
  }
  
  .search-bar {
    padding-left: 1%;
    padding-right: 1%;
    padding-bottom: 2%;
  }
  
  .search-container {
    background-color: #282a36!important;
    margin-bottom: 40px;
  }
  
  .comp-full-calendar {
    margin-bottom: 10%;
  }
  
  .information-modal {}
  
  .form-control {
    color: #8be9fd;
    background: #44475a;
    border-color: transparent;
  }
  
  .form-control:hover,
  .form-control:focus {
    border-top: transparent;
    border-left: transparent;
    border-right: transparent;
    border-bottom: 2px solid #8be9fd;
  }
  
  .infomation-modal-content {}
  
  .map-container {
    height: 70vh;
    width: 100%;
    padding-right: 3%;
    padding-left: 3%;
  }
  
  .app-btn {
    background: transparent;
    border-color: #8be9fd;
    color: #8be9fd;
  }
  
  .app-btn:hover,
  .app-btn:active {
    background: transparent;
    border-color: #bd93f9;
    color: #bd93f9;
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