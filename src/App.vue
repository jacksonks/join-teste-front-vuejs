<template>
  <v-app>
    <v-app-bar
        app
        color="cyan lighten-1"
        dark
    >
      <div>
        <v-img
            alt="Join White Logo"
            contain
            src="https://jointecnologia.com.br/wp-content/themes/theme-bones-master/library/images/logo.png"
            width="125"
        />
      </div>
      <v-spacer></v-spacer>
      <v-btn href="https://github.com/jacksonks/join-teste-front-vuejs" target="_blank" text>
        <span class="mr-2">Github</span>
        <v-icon>code</v-icon>
      </v-btn>
    </v-app-bar>
    <v-main>
      <filters @stations="getStations" @types="getTypes" />

          <v-card width="500" class="text-center popup-container">
            <v-card-title class="headline grey lighten-2">Dados da Estação</v-card-title>
            <v-card-text class="popup-container">
              <v-container fluid class="popup-container">
                <v-row align="center" class="popup-container">
                  <v-col class="d-flex" cols="4">
                  <label>Identificador:</label><strong class="overlay-text" id="feature-id"></strong><br/>
                  </v-col>
                  <v-col class="d-flex" cols="8">
                      <v-btn outlined color="black">
                        <label>Nome: </label><strong class="overlay-text" id="feature-name"></strong>
                      </v-btn>
                    <div style="border-color: black; border-radius: 2px; ">
<!--                  <label>Nome:</label><strong class="overlay-text" id="feature-name"></strong>-->
                    </div>
                  </v-col>
                  <v-col class="d-flex" cols="4">
                    <v-text-field dense hide-details readonly outlined label="Identificador"></v-text-field>
                  </v-col>
                  <v-col class="d-flex" cols="8">
                    <v-text-field dense hide-details readonly outlined label="Nome"></v-text-field>
                  </v-col>
                  <v-col class="d-flex" cols="6">
                    <v-text-field dense hide-details readonly outlined label="Latitude"></v-text-field>
                  </v-col>
                  <v-col class="d-flex" cols="6">
                    <v-text-field dense hide-details readonly outlined label="Longitude"></v-text-field>
                  </v-col>
                  <v-col class="d-flex" cols="4">
                    <v-text-field dense hide-details readonly outlined label="Elevação (m2)"></v-text-field>
                  </v-col>
                  <v-col class="d-flex" cols="4">
                    <v-text-field dense hide-details readonly outlined label="Inicio de Operação"></v-text-field>
                  </v-col>
                  <v-col class="d-flex" cols="4">
                    <v-text-field dense hide-details readonly outlined label="Fim de Operação"></v-text-field>
                  </v-col>
                  <v-col class="d-flex" cols="12">
                    <v-text-field dense hide-details readonly outlined label="Tipo de Estação"></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>
          </v-card>
      <v-card>
        <div id="map"></div>
      </v-card>
    </v-main>
  </v-app>
</template>

<script>
/* eslint-disable */
// import openlayer css for style
import "ol/ol.css";
// This is library of openlayer for handle map
import GeoJSON from 'ol/format/GeoJSON';
import Map from "ol/Map";
import View from "ol/View";
import { defaults as defaultControls, ScaleLine } from "ol/control";
import {Tile as TileLayer, Vector as VectorLayer} from 'ol/layer';
import {OSM, Vector as VectorSource} from 'ol/source';
import {Fill, Stroke, Style, Text, Circle} from 'ol/style';
import Overlay from 'ol/Overlay';
import Filters from "@/components/filters";
export default {
  name: 'App',
  components: {Filters},
  data: () => ({
    id: undefined,
    creat_at: undefined,
    update_At: undefined,
    name: undefined,
    latitude: undefined,
    longitude: undefined,
    elevation_meters: undefined,
    operation_start_date: undefined,
    operation_end_date: undefined,
    station_type_id: undefined,
    dialog: false,
    types: undefined,
    stations: undefined,
    url: 'https://raw.githubusercontent.com/jacksonks/geojson/master/station_list.geojson',
  }),
  async mounted() {
    await this.initiateMap();
  },
  computed:{},
  watch:{},
  methods: {
    getTypes(types){
      console.log('types:', types)
      this.types = types
    },
    async getStations(stations){
      console.log('stations:', stations)
      this.stations = stations
    },
    initiateMap() {
      var fillStyle = new Fill({
        color: 'rgba(255, 255, 255, 0.6)',
      })
      var strokeStyle = new Stroke({
        color: '#319FD3',
        width: 1,
      })
      var circleStyle = new Circle({
        fill: new Fill({
          color: 'rgba(255, 255, 255, 0.6)',
        }),
        radius: 15,
        stroke: strokeStyle,
      })
      // create vector layer
      var source = new VectorSource();
      var vector = new VectorLayer({
        source: source,
      });
      // create title layer
      var raster = new TileLayer({
        source: new OSM()
      });
      // Vector data source in GeoJSON format
      const vectorGeoJSON = new VectorLayer({
        source: new VectorSource({
          format: new GeoJSON(),
          url: this.url,
        }),
        visible: true,
        style: new Style({
          fill: fillStyle,
          stroke: strokeStyle,
          image: circleStyle,
        })
        /*        style: function (feature) {
                  style.getText().setText(feature.get('name'));
                  return style;
                },*/
      })
      // create map with 2 layer
      var map = new Map({
        controls: defaultControls().extend([
          new ScaleLine({
            units: "degrees",
          }),
        ]),
        target: "map",
        layers: [raster, vector,vectorGeoJSON],
        view: new View({
          projection: "EPSG:4326",
          center: [-51.815011395380765,-24.650150016322684],
          zoom: 7,
        }),
      });
      var popup = document.querySelector('.popup-container')
      var overlayLayer =  new Overlay({ element: popup })
      map.addOverlay(overlayLayer)
      var overlayFeatureId = document.getElementById('feature-id')
      var overlayFeatureName = document.getElementById('feature-name')
      map.on('click', function (e){
        overlayLayer.setPosition(undefined)
        map.forEachFeatureAtPixel(e.pixel, function (feature, layer){
          console.log(feature.get('id'))
          console.log(feature.get('name'))
          overlayLayer.setPosition(e.coordinate)
          overlayFeatureId.innerHTML = feature.get('id')
          overlayFeatureName.innerHTML = feature.get('name')
        })
      })
    },
  },
};
</script>
<style>
#map {
  position: absolute;
  padding: 0;
  margin-top: 5px;
  height: 600px;
  width: 100%;
}
</style>