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
      <v-card>
        <div id="map"></div>
      </v-card>
    </v-main>
    <v-card width="500" class="text-center popup-container">
      <v-card-title class="headline grey lighten-2">Dados da Estação</v-card-title>
      <v-card-text class="popup-container">
        <v-container fluid class="popup-container">
          <v-row align="center" class="popup-container mb-n7">
            <v-col class="d-flex" cols="12">
              <v-btn outlined color="black">
                <label>Identificador: </label><strong class="overlay-text" id="feature-id"></strong><br/>
              </v-btn>
              <v-spacer/>
              <v-btn outlined color="black">
                <label>Nome: </label><strong class="overlay-text" id="feature-name"></strong>
              </v-btn>
            </v-col>
            <v-col class="d-flex" cols="12">
              <v-btn outlined color="black" block>
                <label>Latitude:</label> <strong class="overlay-text" id="feature-latitude"></strong>
              </v-btn>
            </v-col>
            <v-col class="d-flex" cols="12">
              <v-btn outlined color="black" block>
                <label>Longitude:</label><strong class="overlay-text" id="feature-longitude"></strong>
              </v-btn>
            </v-col>
            <v-col class="d-flex" cols="12">
              <v-btn outlined color="black">
                <label>Elevação:</label><strong class="overlay-text" id="feature-elevation"></strong>
              </v-btn>
              <v-spacer/>
              <v-btn outlined color="black">
                <label>Tipo de Estação:</label><strong class="overlay-text" id="feature-type"></strong>
              </v-btn>
            </v-col>
            <v-col class="d-flex" cols="12">
              <v-btn outlined color="black" block>
                <label>Inicio de Operação:</label><strong class="overlay-text" id="feature-start"></strong>
              </v-btn>
            </v-col>
            <v-col class="d-flex" cols="12">
              <v-btn outlined color="black" block>
                <label>Fim de Operação:</label><strong class="overlay-text" id="feature-end"></strong>
              </v-btn>
            </v-col>
          </v-row>
        </v-container>
      </v-card-text>
    </v-card>
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
import {Fill, Stroke, Style, Icon, Circle} from 'ol/style';
import Overlay from 'ol/Overlay';
import Filters from "@/components/filters";
export default {
  name: 'App',
  components: {Filters},
  data: () => ({
    station_type_id: undefined,
    dialog: false,
    types: undefined,
    stations: undefined,
    url: 'https://raw.githubusercontent.com/jacksonks/geojson/master/station_list.geojson'
  }),
  async mounted() {
    await this.initiateMap();
  },
  computed:{},
  watch:{},
  methods: {
    getTypes(types){
      console.log('types filtered:', types)
      this.types = types
    },
    async getStations(stations){
      console.log('stations filtered:', stations)
      this.stations = stations
      this.url = stations
      await this.initiateMap();
    },
    initiateMap() {
      var imageStyle = new Icon({
        scale: 1.0,
        color: 'grey',
        src: require('../src/assets/location_on-white-48dp.svg'),
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
          image: imageStyle,
        })
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
      let station_type = [
        {
          "id": "1",
          "created_at": "2020-11-30 10:43:46.687141-03",
          "update_at": "2020-11-30 10:43:46.687162-03",
          "name": "AGROMETEOROLOGICAL",
          "color": "#7cb5ec"
        },
        {
          "id": "2",
          "created_at": "2020-11-30 10:43:46.688442-03",
          "update_at": "2020-11-30 10:43:46.688459-03",
          "name": "CLIMATOLÓGICO",
          "color": "#434348"
        },
        {
          "id": "3",
          "created_at": "2020-11-30 10:43:46.68895-03",
          "update_at": "2020-11-30 10:43:46.688964-03",
          "name": "HIDROCLIMATOLÓGICO",
          "color": "#90ed7d"
        },
        {
          "id": "4",
          "created_at": "2020-11-30 10:43:46.689495-03",
          "update_at": "2020-11-30 10:43:46.689513-03",
          "name": "HIDROMÉTRICO",
          "color": "#f7a35c"
        },
        {
          "id": "5",
          "created_at": "2020-11-30 10:43:46.690113-03",
          "update_at": "2020-11-30 10:43:46.690134-03",
          "name": "PLUVIOMETRIC",
          "color": "#8085e9"
        }
      ]
      var overlayFeatureId = document.getElementById('feature-id')
      var overlayFeatureName = document.getElementById('feature-name')
      var overlayFeatureLatitude = document.getElementById('feature-latitude')
      var overlayFeatureLongitude = document.getElementById('feature-longitude')
      var overlayFeatureElevation = document.getElementById('feature-elevation')
      var overlayFeatureType = document.getElementById('feature-type')
      var overlayFeatureStart = document.getElementById('feature-start')
      var overlayFeatureEnd = document.getElementById('feature-end')
      map.on('click', function (e){
        overlayLayer.setPosition(undefined)
        map.forEachFeatureAtPixel(e.pixel, function (feature, layer){
          overlayLayer.setPosition(e.coordinate)
          overlayFeatureId.innerHTML = feature.get('id')
          overlayFeatureName.innerHTML = feature.get('name')
          overlayFeatureLatitude.innerHTML = feature.get('latitude')
          overlayFeatureLongitude.innerHTML = feature.get('longitude')
          overlayFeatureElevation.innerHTML = feature.get('elevation_meters')
          overlayFeatureStart.innerHTML = feature.get('operation_start_date')
          overlayFeatureEnd.innerHTML = feature.get('operation_end_date')
          for(let item in station_type){
            if(feature.get('station_type_id') == station_type[item].id){
              overlayFeatureType.innerHTML = station_type[item].name
            }
          }
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
  margin-top: 0px;
  height: 600px;
  width: 100%;
}
</style>