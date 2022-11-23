<template>
  <div class="ui grid">
    <div class="six wide column red">
      <form class="ui segment large form">
        <div class="ui message red" v-show="error">{{ error }}</div>
        <div class="ui segment">
          <div class="field">
            <div
              class="ui right icon input large"
              :class="{ loading: spinner }"
            >
              <input
                type="text"
                placeholder="Enter your address"
                v-model="address"
                id="autocomplete"
              />
              <i class="dot circle link icon" @click="locatorButtonPressed"></i>
            </div>
          </div>
          <div class="field">
            <div class="two fields">
              <div class="field">
                <select v-model="type">
                  <option value="restaurant">Restaurant</option>
                </select>
              </div>

              <div class="field">
                <select v-model="radius">
                  <option value="3">5 MI</option>
                  <option value="6">10 MI</option>
                  <option value="9">15 MI</option>
                  <option value="12">20 MI</option>
                </select>
              </div>
            </div>
          </div>

          <button class="ui button" @click="findCloseBuyButtonPressed">
            Find CloseBuy Places
          </button>
        </div>
      </form>

      <div class="ui segment" style="max-height:500px;overflow:auto;">
        <div class="ui divided items">
          <div class="item" v-for="place in places" :key="place.id">
            <div class="content">
              <div class="header">{{ place.name }}</div>
              <div class="meta">{{ place.vicinity }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="ten wide column blue" ref="map"></div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      address: "",
      error: "",
      spinner: false,
      apiKey: "AIzaSyCTzd3EuoCv43xqDh43u65f0s6Jk5kjK_4",
      lat: 0,
      lng: 0,
      type: "",
      radius: "",
      places: []
    };
  },

  mounted() {
    new google.maps.places.Autocomplete(this.$refs["autocomplete"], {
      bounds: new google.maps.LatLngBounds(
        new google.maps.LatLng(33.10317, -96.67055)
      )
    });
  },

  methods: {
    locatorButtonPressed() {
      this.spinner = true;
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          position => {
            (this.lat = position.coords.latitude),
              (this.lng = position.coords.longitude);

            this.getAddressFrom(
              position.coords.latitude,
              position.coords.longitude
            );
            this.showUserLocationOnTheMap(
              position.coords.latitude,
              position.coords.longitude
            );
          },
          error => {
            this.error =
              "Locator is unable to find your address. Please type your address manually.";
            this.spinner = false;
            // console.log(error.message);
          }
        );
      } else {
        this.error = error.message;
        this.spinner = false;
        console.log("Your browser does not support geolocation API");
      }
    },
    getAddressFrom(lat, long) {
      axios
        .get(
          "https://maps.googleapis.com/maps/api/geocode/json?latlng=" +
            lat +
            "," +
            long +
            "&key=" +
            this.apiKey
        )
        .then(response => {
          if (response.data.error_message) {
            this.error = response.data.error_message;
            console.log(response.data.error_message);
          } else {
            this.address = response.data.results[0].formatted_address;
          }
          this.spinner = false;
        })
        .catch(error => {
          this.error = error.message;
          console.log(error.message);
        });
    },
    showUserLocationOnTheMap(latitude, longitude) {
      // Create A Map Object
      let map = new google.maps.Map(document.getElementById("map"), {
        zoom: 15,
        center: new google.maps.LatLng(latitude, longitude),
        mapTypeId: google.maps.MapTypeId.ROADMAP
      });
      // Add Marker
      new google.maps.Marker({
        position: new google.maps.LatLng(latitude, longitude),
        map: map
      });
    },
    findCloseBuyButtonPressed() {
      const URL = `https://cors-anywhere.herokuapp.com/https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=${
        this.lat
      },${this.lng}&type=${this.type}&radius=${this.radius * 1000}&key=${
        this.apiKey
      }`;

      axios
        .get(URL)
        .then(response => {
          this.places = response.data.results;
          this.showPlacesOnMap();
          console.log(response);
        })
        .catch(error => {
          this.error = error.message;
        });
    },
    showPlacesOnMap() {
      const map = new google.maps.Map(this.$refs["map"], {
        zoom: 15,
        center: new google.maps.LatLng(this.lat, this.lng),
        mapTypeId: google.maps.MapTypeId.ROADMAP
      });

      const infoWindow = new google.maps.InfoWindow();

      for (let i = 0; i < this.places.length; i++) {
        const lat = this.places[i].geometry.location.lat;
        const lng = this.places[i].geometry.location.lng;

        const marker = new google.maps.Marker({
          position: new google.maps.LatLng(lat, lng),
          map: map
        });


        google.maps.event.addListener(marker, "click", () => {
          infoWindow.setContent(`<div class="ui header">${this.places[i].name}</div>`);
          infoWindow.open(map, marker);
        });
      }
    }
  }
};
</script>

<style>
.ui.button,
.dot.circle.icon {
  background-color: #ff5a5f;
  color: white;
}

.pac-icon {
  display: none;
}

.pac-item {
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
}

.pac-item:hover {
  background-color: #ececec;
}

.pac-item-query {
  font-size: 16px;
}

#map {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}
</style>
