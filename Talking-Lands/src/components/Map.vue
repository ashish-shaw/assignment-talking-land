<template>
    <div class="mt-10">
        <div class="text-xl font-bold text-gray-800 text-center pb-5">Map with Multiple Marker</div>
        <div id="mapContainer" @click="popupDetails"></div>
    </div>
   
</template>
<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import { useGeolocation } from "@vueuse/core";


const { coords } = useGeolocation();

export default {
  name: "Map",
  data() {
    return {
      center: {
        latitude: 12.972442,
        longitude: 77.580643,
      },
      nearbyMarkers: [],
      popup: null,
      mapDiv: null,
    };
  },
  methods: {
    setupLeafletMap() {
      this.mapDiv = L.map("mapContainer").setView(
        [this.center.latitude, this.center.longitude],
        13
      );
      L.tileLayer(
        "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}",
        {
          attribution:
            '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
          maxZoom: 18,
          id: "mapbox/streets-v11",
          accessToken:
            "pk.eyJ1IjoiYXNoaXNoLTk4IiwiYSI6ImNscmlraWk4bzBibTcybHFwcjFtYWdmZHYifQ.OoE7tQpq5v2yRVkOY_xS2w",
        }
      ).addTo(this.mapDiv);
      var marker = L.marker([this.center.latitude, this.center.longitude])
        .addTo(this.mapDiv)
        .bindPopup(`The Latitude and Longitude of this place is (<strong>${this.center.latitude.toFixed(
              2
            )},${this.center.longitude.toFixed(2)}</strong>)`);

      this.nearbyMarkers.forEach(({ latitude, longitude }) => {
        L.marker([latitude, longitude])
          .addTo(this.mapDiv)
          .bindPopup(
            `The Latitude and Longitude of this place is (<strong>${latitude.toFixed(
              2
            )},${longitude.toFixed(2)}</strong>)`
          );
      });

      this.mapDiv.addEventListener("click", (e) => {
        const { lat: latitude, lng: longitude } = e.latlng;

        L.marker([latitude, longitude])
          .addTo(this.mapDiv)
          .bindPopup(
            `The Latitude and Longitude of this place is (<strong>${latitude.toFixed(
              2
            )},${longitude.toFixed(2)}</strong>)`
          );

        this.nearbyMarkers.push({ latitude, longitude });
      });
    },
  },
  mounted() {
    this.setupLeafletMap();
  },

  watch() {
    if (
      coords.value.latitude !== Number.POSITIVE_INFINITY &&
      coords.value.longitude !== Number.POSITIVE_INFINITY
    ) {
      this.center.latitude = coords.value.latitude;
      this.center.longitude = coords.value.longitude;

      if (userGeoMarker) {
        this.mapDiv.removeLayer(userGeoMarker);
      }

      userGeoMarker = leaflet
        .marker([this.center.latitude, this.center.longitude])
        .addTo(this.mapDiv)
        .bindPopup("User Marker");

      this.mapDiv.setView([this.center.latitude, this.center.longitude], 13);

      const el = userGeoMarker.getElement();
      if (el) {
        el.style.filter = "hue-rotate(120deg)";
      }
    }
  },
};
</script>

<style scoped>
#mapContainer {
  width: 80vh;
  height: 90vh;
}
</style>
