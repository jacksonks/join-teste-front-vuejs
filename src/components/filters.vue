<template>
  <v-container fluid>
    <v-row class="text-center">
      <v-col col="6">
      <v-col cols="12">
        <v-img
            alt="Join Blue Logo"
            :src="require('../assets/logo-blue.png')"
            class="my-3"
            contain
            height="100"
        />
      </v-col>

      <v-col class="mb-4">
        <h1 class="display-1 font-weight-bold mb-3">
          Mapa para Visualização de Localização de Estações
        </h1>

        <p class="subheading font-weight-bold">
          Desenvolvido com
          <a
              href="https://vuejs.org/"
              target="_blank"
          >VueJS</a>,
          <a
              href="https://vuetifyjs.com/en/"
              target="_blank"
          >Vuetify</a>, e
          <a
              href="https://openlayers.org/"
              target="_blank"
          >OpenLayers</a>,
          por
          <a
              href="https://www.linkedin.com/in/jackson-kelvin/"
              target="_blank"
          >Jackson Kelvin de Souza</a>.
        </p>
      </v-col>
      </v-col>

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
                    :items="station_list"
                    return-object
                    item-text="station_type_id"
                    label="Tipos de Estações"
                    multiple
                    outlined
                    chips
                    clearable
                    persistent-hint
                    :hint="`${stations_types_selected.length} Opções Selecionadas`"
                >
                  <template v-slot:selection="data">
                    <v-chip
                        :key="JSON.stringify(data.item)"
                        v-bind="data.attrs"
                        :input-value="data.selected"
                        :disabled="data.disabled"
                        @click:close="data.parent.selectItem(data.item)"
                    >
                      {{ data.item.station_type_id }}
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
                  <template v-slot:prepend-item>
                    <v-list-item ripple @click="toggle">
                      <v-list-item-action>
                        <v-icon color="indigo darken-4">{{ icon }}</v-icon>
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
    station_list: undefined,
    stations_types_selected: [],
    stations_selected: [],
    expand: true,
  }),
  created() {
    this.fetchApi();
  },
  computed:{
    stations(){
      let aux = []
      if(this.stations_types_selected.length !== 0){
        for(let station in this.stations_types_selected){
          let filtered = this.station_list.filter(a => {
            return a.station_type_id === this.stations_types_selected[station].station_type_id;
          });
          for(let item in filtered){ aux.push(filtered[item])}
        }
        return aux
      } else { return undefined }
    },
    all() {
      return this.stations_selected.length === this.stations.length
    },
    some () {
      return this.stations_selected.length > 0 && !this.all
    },
    icon () {
      if (this.all) return 'disabled_by_default'
      if (this.some) return 'indeterminate_check_box'
      return 'check_box_outline_blank'
    },
  },
  methods: {
    toggle () {
      this.$nextTick(() => {
        if (this.all) {
          this.stations_selected = []
        } else {
          this.stations_selected = this.stations.slice()
        }
      })
    },
    fetchApi() {
      fetch("http://localhost:3000/station_list")
          .then(response => response.json())
          .then(response => {
            this.station_list = response;
          })
          .catch((error) => {
            console.log('Looks like there was a problem: \n', error);
          });
    },
    consulting(){
      let parameter = this.stations_selected
      this.$emit('parameter', parameter)
    },
  },
}
</script>