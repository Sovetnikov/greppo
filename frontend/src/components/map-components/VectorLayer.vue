<template>
  <l-marker-cluster>
    <l-geo-json
        :name="layerData.name"
        :geojson="layerData.data"
        :options="options"
        :visible="getLayerVisibility.bind(this, layerData.id)()"
        layer-type="overlay"
    ></l-geo-json>
  </l-marker-cluster>
</template>

<script>
import {LGeoJson} from "vue2-leaflet";
import Vue2LeafletMarkercluster from 'vue2-leaflet-markercluster';
import {mapGetters} from "vuex";
import "leaflet.markercluster/dist/MarkerCluster.css";
import "leaflet.markercluster/dist/MarkerCluster.Default.css";

export default {
    name: "VectorLayer",
    props: {
        layerData: Object,
    },
    components: {
        'l-geo-json': LGeoJson,
        'l-marker-cluster': Vue2LeafletMarkercluster,
    },
    methods: {
        getChoroplethColor(d) {
            const index = this.layerData.style.choropleth.bins.findIndex(
                (item) => d > item
            );
            const color = this.layerData.style.choropleth.palette[index];
            return color;
        },
    },
    computed: {
        ...mapGetters(["getLayerVisibility"]),
        clusterOptions() {
          return {disableClusteringAtZoom: 11 };
        },
        options() {
            return {
                onEachFeature: this.onEachFeatureFunction,
                pointToLayer: this.pointToLayerFunction,
                style: this.styleFunction,
            };
        },
        pointToLayerFunction() {
            return (feature, latlng) => {
                return window.L.circleMarker(latlng);
            };
        },
        styleFunction() {
            return (feature) => {
                return {
                    stroke: this.layerData.style.stroke || true,
                    color:
                        this.layerData.style.color ||
                        this.layerData.style.fillColor,
                    weight: this.layerData.style.weight || 2,
                    opacity: this.layerData.style.opacity || 1,
                    lineCap: this.layerData.style.lineCap || "round",
                    lineJoin: this.layerData.style.lineJoin || "round",
                    dashArray: this.layerData.style.dashArray || null,
                    dashOffset: this.layerData.style.dashOffset || null,
                    fillColor: this.layerData.style.choropleth
                        ? this.getChoroplethColor(
                              feature.properties[
                                  this.layerData.style.choropleth.key_on
                              ]
                          )
                        : this.layerData.style.fillColor,
                    fillOpacity: this.layerData.style.fillOpacity || 0.8,
                    radius: this.layerData.style.radius || 3,
                };
            };
        },
        onEachFeatureFunction() {
            return (feature, layer) => {
                if (feature.properties) {
                    var bindContent = "<div>";
                    for (var p in feature.properties) {
                        bindContent +=
                            '<p style="margin: 5px 10px;"><span style="font-weight:500;">' +
                            p +
                            "</span>:  " +
                            feature.properties[p] +
                            "</p>";
                    }
                    bindContent += "</div>";
                    // Bind a popup
                    // layer.bindPopup(bindContent);
                    // Bind a tooltip
                    layer.bindTooltip(bindContent, {
                        permanent: false,
                        sticky: true,
                    });
                }
            };
        },
    },
};
</script>

<style></style>
