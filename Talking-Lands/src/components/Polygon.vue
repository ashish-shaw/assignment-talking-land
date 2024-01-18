<template>
  <div class="mt-10">
    <div class="text-xl font-bold text-gray-800 text-center pb-5">
      Polygon Map
    </div>
    <div :id="mapId"></div>
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import { statesData } from "@/store/state";

export default {
  name: "Polygon",
  data() {
    return {
      center: [37.8, -96],
      mapId: "polygon",
      info: null,
      geoJson: null,
      mapDiv: null,
    };
  },
  methods: {
    setupLeafletMap() {
       this.mapDiv = L.map(this.mapId).setView(this.center, 4);

      L.tileLayer(
        "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}",
        {
          attribution:
            '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
          maxZoom: 19,
          id: "mapbox/streets-v11",
          accessToken:
            "pk.eyJ1IjoiYXNoaXNoLTk4IiwiYSI6ImNscmlraWk4bzBibTcybHFwcjFtYWdmZHYifQ.OoE7tQpq5v2yRVkOY_xS2w",
        }
      ).addTo(this.mapDiv);

      L.geoJson(statesData, {
        style: this.style,
        onEachFeature: this.onEachFeature,
      }).addTo(this.mapDiv);

      this.info = L.control();
      this.info.onAdd = function (map) {
        this._div = L.DomUtil.create("div", "info");
        this.update();
        return this._div;
      };

      this.info.update = function (props) {
        const contents = props
          ? `<b>${props.name}</b><br />${props.density} people / mi<sup>2</sup>`
          : "Hover over a state";
        this._div.innerHTML = `<div class="bg-white p-2"><h4>US Population Density</h4>${contents}</div>`;
      };

      this.info.addTo(this.mapDiv);

      const legend = L.control({ position: "bottomright" });
      legend.onAdd = function (map) {
        const div = L.DomUtil.create("div", "info legend");
        const grades = [0, 10, 20, 50, 100, 200, 500, 1000];
        const labels = [];
        let from, to;

        for (let i = 0; i < grades.length; i++) {
          from = grades[i];
          to = grades[i + 1];

          labels.push(
            `<i style="background:${this.getColor(from + 1)}"></i> ${from}${
              to ? `&ndash;${to}` : "+"
            }`
          );
        }

        div.innerHTML = labels.join("<br>");
        return div;
      };

      legend.addTo(this.mapDiv);
    },
    getColor(d) {
      return d > 1000
        ? "#800026"
        : d > 500
        ? "#BD0026"
        : d > 200
        ? "#E31A1C"
        : d > 100
        ? "#FC4E2A"
        : d > 50
        ? "#FD8D3C"
        : d > 20
        ? "#FEB24C"
        : d > 10
        ? "#FED976"
        : "#FFEDA0";
    },
    style(feature) {
      return {
        weight: 2,
        opacity: 1,
        color: "white",
        dashArray: "3",
        fillOpacity: 0.7,
        fillColor: this.getColor(feature.properties.density),
      };
    },
    highlightFeature(e) {
      const layer = e.target;

      layer.setStyle({
        weight: 5,
        color: "#666",
        dashArray: "",
        fillOpacity: 0.7,
      });

      layer.bringToFront();

      this.info.update(layer.feature.properties);
    },
    resetHighlight(e) {
      const geoJson = L.geoJson(statesData, {
        style: this.style,
        onEachFeature: this.onEachFeature,
      }).addTo(this.mapDiv);
      geoJson.resetStyle(e.target);
      this.info.update();
    },
    zoomToFeature(e) {
      this.map.fitBounds(e.target.getBounds());
    },
    onEachFeature(feature, layer) {
      layer.on({
        mouseover: this.highlightFeature,
        mouseout: this.resetHighlight,
        click: this.zoomToFeature,
      });
    },
  },
  mounted() {
    this.setupLeafletMap();
  },
};
</script>

<style scoped>
#polygon {
  width: 85vh;
  height: 90vh;
}
</style>
