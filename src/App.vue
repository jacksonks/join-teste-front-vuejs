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
      <v-card width="500" class="text-center popup-container">
        <v-card-title class="headline grey lighten-2">Dados da Estação</v-card-title>
        <v-card-text class="popup-container">
          <v-container fluid class="popup-container">
            <v-row align="center" class="popup-container">
              <v-col class="d-flex" cols="12">
                <v-btn outlined color="black">
                  <label>Identificador:</label><strong class="overlay-text" id="feature-id"></strong><br/>
                </v-btn>
                <v-spacer/>
                <v-btn outlined color="black">
                  <label>Nome: </label><strong class="overlay-text" id="feature-name"></strong>
                </v-btn>
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
    <filters @stations="getStations" @types="getTypes" />
    <v-footer>
      <v-container fluid>
        <v-row class="text-center">
          <v-col cols="12">
            <v-img alt="Join Blue Logo" :src="require('../src/assets/logo-blue.png')" class="my-3" contain height="100"/>
          </v-col>
          <v-col class="mb-4">
            <h1 class="display-1 font-weight-bold mb-3">Mapa para Visualização de Localização de Estações</h1>
            <p class="subheading font-weight-bold">Desenvolvido com
              <a href="https://vuejs.org/" target="_blank">VueJS</a>,
              <a href="https://vuetifyjs.com/en/" target="_blank">Vuetify</a>, e
              <a href="https://openlayers.org/" target="_blank">OpenLayers</a>, por
              <a href="https://www.linkedin.com/in/jackson-kelvin/" target="_blank">Jackson Kelvin de Souza</a>.
            </p>
          </v-col>
        </v-row>
      </v-container>
    </v-footer>
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
      var colors = [
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
      var imageStyle = new Icon({
        color: 'red',
        scale: 0.7,
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
          url: 'https://raw.githubusercontent.com/jacksonks/geojson/master/station_list.geojson',
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