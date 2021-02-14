<template>
  <v-container fluid>
    <v-row class="text-center">
      <v-col cols="6">
        <v-container fluid>
          <v-row>
            <v-card width="600">
              <v-card-title class="headline grey lighten-2">
                Filtros
                <v-spacer/>
                <v-tooltip right>
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn
                        v-bind="attrs"
                        v-on="on"
                        text
                        fab
                        @click="expand = !expand"
                    >
                      <v-icon large v-if="expand === true">compress</v-icon>
                      <v-icon large v-else>expand</v-icon>
                    </v-btn>
                  </template>
                  <strong v-if="expand === true">Esconder Filtros</strong>
                  <strong v-else>Mostrar Filtros</strong>
                </v-tooltip>
              </v-card-title>
              <v-card-text v-show="expand">
                <br/>
                <v-select
                    v-model="stations_types_selected"
                    :items="station_type_list"
                    return-object
                    item-text="name"
                    label="Tipos de Estações"
                    multiple
                    outlined
                    chips
                    clearable
                    persistent-hint
                    :hint="`${stations_types_selected.length} Opções Selecionadas`"
                >

                  <template v-slot:prepend-item>
                    <v-list-item ripple @click="change">
                      <v-list-item-action>
                        <v-icon color="indigo darken-4">{{ iconTypes }}</v-icon>
                      </v-list-item-action>
                      <v-list-item-content>
                        <v-list-item-title>
                          Selecionar Todos
                        </v-list-item-title>
                      </v-list-item-content>
                    </v-list-item>
                    <v-divider/>
                  </template>

                  <template v-slot:selection="data">
                    <v-chip
                        :key="JSON.stringify(data.item)"
                        v-bind="data.attrs"
                        :input-value="data.selected"
                        :disabled="data.disabled"
                        @click:close="data.parent.selectItem(data.item)"
                    >
                      {{ data.item.name }}
                    </v-chip>
                  </template>
                </v-select>
                <v-select
                    v-model="stations_selected"
                    :items="stations"
                    return-object
                    item-key="id"
                    item-text="name"
                    label="Estações"
                    multiple
                    outlined
                    chips
                    clearable
                    :disabled="!stations_types_selected.length > 0"
                    :hide-details="!stations_types_selected.length > 0"
                    :persistent-hint="stations_types_selected.length > 0"
                    :hint="`${stations_selected.length} Opções Selecionadas`"
                >
                  <template v-slot:no-data>
                    <v-list-item>
                      <strong style="color: red">*Sem Estações do Tipo Selecionado</strong>
                    </v-list-item>
                  </template>

                  <template v-slot:prepend-item>
                    <v-list-item ripple @click="toggle">
                      <v-list-item-action>
                        <v-icon color="indigo darken-4">{{ iconStations }}</v-icon>
                      </v-list-item-action>
                      <v-list-item-content>
                        <v-list-item-title>
                          Selecionar Todos
                        </v-list-item-title>
                      </v-list-item-content>
                    </v-list-item>
                    <v-divider/>
                  </template>

                  <template v-slot:selection="data">
                    <v-chip
                        :key="JSON.stringify(data.item)"
                        v-bind="data.attrs"
                        :input-value="data.selected"
                        :disabled="data.disabled"
                        @click:close="data.parent.selectItem(data.item)"
                    >
                      {{ data.item.name }}
                    </v-chip>
                  </template>
                </v-select>
                <span v-show="!stations_types_selected.length > 0" style="color: red">*selecione pelo menos 1 tipo de estação para desbloquear o seletor de estações</span>
              </v-card-text>
              <v-divider></v-divider>
              <v-card-actions>
                <v-spacer/>
                <v-btn color="primary" @click="consulting">Consultar<v-icon right>search</v-icon></v-btn>
                <v-spacer/>
              </v-card-actions>
            </v-card>
          </v-row>
        </v-container>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
/* eslint-disable */
export default {
  data: () => ({
    station_list: [],
    station_type_list: [],
    stations_types_selected: [],
    stations_selected: [],
    expand: true,
    geojson: undefined,
  }),
  created() {
    this.fetchStation();
    this.fetchStationType();
    this.fetchFeatures();
  },
  computed:{
    stations(){
      let aux = []
      if(this.stations_types_selected.length !== 0){
        for(let station in this.stations_types_selected){
          let filtered = this.station_list.filter(a => {
            return a.station_type_id === this.stations_types_selected[station].id;
          });
          for(let item in filtered){ aux.push(filtered[item])}
        }
        return aux
      } else { return [] }
    },
    allTypes() {
      return this.stations_types_selected.length === this.station_type_list.length
    },
    someTypes () {
      return this.stations_types_selected.length > 0 && this.stations_types_selected.length < this.station_type_list.length
    },
    emptyTypes (){
      return this.stations_types_selected.length === 0
    },
    allStations() {
      return this.stations_selected.length === this.stations.length
    },
    someStations () {
      return this.stations_selected.length > 0 && this.stations_selected.length < this.stations.length
    },
    emptyStations (){
      return this.stations_selected.length === 0
    },
    iconStations () {
      if (this.allStations) return 'disabled_by_default'
      if (this.someStations) return 'indeterminate_check_box'
      if (this.emptyStations) return 'check_box_outline_blank'
    },
    iconTypes () {
      if (this.allTypes) return 'disabled_by_default'
      if (this.someTypes) return 'indeterminate_check_box'
      if (this.emptyTypes) return 'check_box_outline_blank'
    },
  },
  methods: {
    change(){
      this.$nextTick(() => {
        if (this.allTypes) {
          this.stations_types_selected = []
        }
        else if(this.someTypes){
          this.stations_types_selected = []
        } else if (this.emptyTypes) {
          this.stations_types_selected = this.station_type_list.slice()
        }
      })
    },
    toggle () {
      this.$nextTick(() => {
        if (this.allStations) {
          this.stations_selected = []
        }
        else if(this.someStations){
          this.stations_selected = []
        } else if (this.emptyStations) {
          this.stations_selected = this.stations.slice()
        }
      })
    },
    fetchFeatures() {
      fetch("https://raw.githubusercontent.com/jacksonks/geojson/master/station_list.geojson")
          .then(response => response.json())
          .then(response => {
            this.geojson = response;
          })
          .catch((error) => {
            console.log('Looks like there was a problem: \n', error);
          });
    },
    fetchStation() {
      fetch("https://raw.githubusercontent.com/jacksonks/geojson/master/station.json")
          .then(response => response.json())
          .then(response => {
            this.station_list = response.station;
          })
          .catch((error) => {
            console.log('Looks like there was a problem: \n', error);
          });
    },
    fetchStationType() {
      fetch("https://raw.githubusercontent.com/jacksonks/geojson/master/station_type.json")
          .then(response => response.json())
          .then(response => {
            this.station_type_list = response.station_type;
          })
          .catch((error) => {
            console.log('Looks like there was a problem: \n', error);
          });
    },
    consulting(){
      //console.log('original:', this.geojson)
      let features = []
      for(let item in this.geojson.features){
        for(let station in this.stations_selected){
          if(this.geojson.features[item].properties.id ===  this.stations_selected[station].id){
            features.push(this.geojson.features[item])
          }
        }
      }
      //console.log('features:', features)
      this.geojson.features = features
      //console.log('geo:', this.geojson)
      this.$emit('stations', this.geojson)
      this.$emit('types', this.stations_types_selected)
    },
  },
}
</script>