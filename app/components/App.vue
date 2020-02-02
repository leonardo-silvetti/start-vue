<template>
    <Page actionBarHidden="true" backgroundSpanUnderStatusBar="true" statusBarStyle="light">
      <StackLayout>
        <SearchBar hint="Cerca..." height="10%" v-model="query" @textChange="handleSearch" @clear="handleClear" />
        <WebView v-if="filePath && op=='map'" height="90%" :src="filePath" row="0" col="0" />
        <ListView v-if="resultList && op=='list'" for="result in resultList" height="90%" @itemTap="handleItemTap">
          <v-template>
            <FlexboxLayout flexDirection="row">
              <Label :text="result.display_name" />
            </FlexboxLayout>
          </v-template>
        </ListView>
      </StackLayout>
    </Page>
</template>

<script >

  import * as http from "http";
  const geoLocation = require("nativescript-geolocation");

  export default {

    data() {
      return {
        op: 'map',
        filePath: null,
        query: null,
        resultList: null,
        currentGeoLocation: {
            latitude: null,
            longitude: null,
            altitude: null,
            direction: null
        }
      }
    },

    mounted() {
      console.log("mounted event fired");
      this.enableLocationServices();
    },

    methods: {
      enableLocationServices: function() {
        geoLocation.isEnabled().then(enabled => {
            if (!enabled) {
              geoLocation
                .enableLocationRequest()
                .then(() => this.showLocation());
            } else {
              this.showLocation();
            }
        });
      },

      showLocation: function() {
        geoLocation.watchLocation(
          location => {
            let baseUrl = '~/www/map.html';
            this.currentGeoLocation = location;
            this.filePath = baseUrl + 
              '?lat=' + this.currentGeoLocation.latitude + 
              '&lng=' + this.currentGeoLocation.longitude;
            console.log(this.filePath);
          },
          error => {
            alert(error);
          }, {
            desiredAccuracy: 3,
            updateDistance: 10,
            minimumUpdateTime: 1000 * 1
          }
        );
      },

      handleSearch: function() {
        if(this.query.length>3) {
          this.op = 'list';
          http.getJSON("https://nominatim.openstreetmap.org/search?format=json&q="+this.query).then(result => {
            this.resultList = result;
          }, error => {
            console.log(error);
          });
        }
      },

      handleItemTap: function(event) {
        this.op = 'map';
        this.resultList = null;
        let baseUrl = '~/www/map.html';
        this.currentGeoLocation.latitude = event.item.lat;
        this.currentGeoLocation.longitude = event.item.lon;
        this.filePath = baseUrl + 
          '?lat=' + this.currentGeoLocation.latitude + 
          '&lng=' + this.currentGeoLocation.longitude;
      },

      handleClear: function() {
        this.op = 'map';
        this.resultList = null;
      }
  },

  }
</script>

<style scoped>
    FlexboxLayout {
      vertical-align: center;
      font-size: 20;
      height: 130px;
    }
</style>
