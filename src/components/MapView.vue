<template>
	<div>
    <app-map-legend class="legend"></app-map-legend>
    <transition name="fade" mode="out-in">
      <app-water-quality-floater class="floater-position" v-if="waterQualityGraph"></app-water-quality-floater>
    </transition>   
    <l-map :style="mapStyleObj" :zoom="zoom" :center="center" ref="ettMap" alt="Data explorer map">
      <l-control-layers position="topleft"></l-control-layers>
      <l-tile-layer
        v-for="tile in basemaps[0]"
        :key="tile.name"
        :name="tile.name"
        :visible="tile.visible"
        :url="tile.url"
        :attribution="tile.attribution"
        :options="tile.options"
        layer-type="base"
      ></l-tile-layer>
      <l-wms-tile-layer
        v-for="tile in basemapsWms"
        :key="tile.name"
        :base-url="tile.url"
        :visible="tile.visible"
        :name="tile.name"
        :attribution="tile.attribution"
        :transparent="tile.transparent"
        :format="tile.format"
        layer-type="base"
      ></l-wms-tile-layer>
      <l-tile-layer
        v-for="tile in basemaps[1]"
        :key="tile.name"
        :name="tile.name"
        :visible="tile.visible"
        :url="tile.url"
        :attribution="tile.attribution"
        :options="tile.options"
        layer-type="base"
      ></l-tile-layer>
      <l-geo-json 
        :geojson="embaymentsGeojson" 
        :options="optionsEmbayment" 
        :options-style="embStyle"
        ref="embaymentsBase"></l-geo-json>
      <div v-if="metricLayer">
        <l-geo-json :geojson="embaymentsGeojson" v-if="habitat=='tidal flats'" :options-style="tidalFlatsColorer" ref="metric"></l-geo-json>	  		
        <l-geo-json :geojson="embaymentsGeojson" v-if="habitat=='salt marsh'" :options-style="saltMarshColorer" ref="metric"></l-geo-json>	  
        <l-geo-json :geojson="embaymentsGeojson" v-if="habitat=='eelgrass'" :options-style="eelgrassColorer" ref="metric"></l-geo-json>	  
      </div>
      <div v-if="baseLayer">
        <l-layer-group ref="historic">
          <l-geo-json :geojson="bkgrdGeojson.tidalFlats.data.historic" v-if="habitat=='tidal flats'" :options-style="tfStyleHis"></l-geo-json>	  		
          <l-geo-json :geojson="bkgrdGeojson.saltMarsh.data.historic" v-if="habitat=='salt marsh'" :options-style="smStyleHis"></l-geo-json>	
          <l-geo-json :geojson="bkgrdGeojson.eelGrass.data.historic" v-if="habitat=='eelgrass'" :options-style="egStyleHis"></l-geo-json>
        </l-layer-group>
        <l-layer-group ref="current">
          <l-geo-json :geojson="bkgrdGeojson.tidalFlats.data.current" v-if="habitat=='tidal flats'" :options-style="tfStyleCur"></l-geo-json>	  		
          <l-geo-json :geojson="bkgrdGeojson.saltMarsh.data.current" v-if="habitat=='salt marsh'" :options-style="smStyleCur"></l-geo-json>	
          <l-geo-json :geojson="bkgrdGeojson.eelGrass.data.current" v-if="habitat=='eelgrass'" :options-style="egStyleCur"></l-geo-json>
        </l-layer-group>
      </div>
      <l-geo-json 
        :geojson="staticGeojson.nonCoastal.data" 
        :options-style="nonCoastalStyle"
        ref="snepRegion">
      </l-geo-json>
      <l-geo-json 
        :geojson="staticGeojson.snepRegion.data" 
        :options-style="regionStyle"
        ref="snepRegion">
      </l-geo-json>
      <div v-if="pointsLayer">
        <l-layer-group ref="stations" v-for="s in stations" :key="s.id">
          <l-circle-marker
            :lat-lng="[s.geometry.coordinates[1], s.geometry.coordinates[0]]"
            :fillColor="circleColorer(s.properties.parameter_list)"
            fillOpacity=1
            :color="activeCircleColorer(s.id, s.properties.parameter_list)"
            radius=5
            @click="plotData($event, s.id, s.properties.parameter_list, s.properties.eda_unit_name)"
          >
            <l-tooltip ref="tooltip" style="padding-left: 15px; padding-right: 15px">
              <app-station-tooltip 
                :stationId="s.id"
                :parameterList="s.properties.parameter_list" 
                :organizationName="s.properties.organization_identifier">
              </app-station-tooltip>
            </l-tooltip>
          </l-circle-marker>
        </l-layer-group>
      </div>
    </l-map>
	</div>
</template>

<script>
import { Icon } from 'leaflet';

delete Icon.Default.prototype._getIconUrl;
Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});
  
import {LMap, LTileLayer, LWMSTileLayer, LLayerGroup, LGeoJson, LCircleMarker, LTooltip, LControlLayers} from 'vue2-leaflet';
import { interpolateGreens, interpolateOranges, interpolatePurples, interpolateBlues } from 'd3-scale-chromatic'
import StationTooltip from './subs/StationTooltip.vue'
import MapLegend from './subs/MapLegend.vue'
import WaterQualityFloater from '../components/WaterQualityFloater.vue'
import { imageLibraryHabitat } from '../lib/constants'
import { basemaps } from '../lib/constants'
import { basemapsWms } from '../lib/constants'
export default {
  components: {
    LMap,
    LTileLayer,
    'l-wms-tile-layer': LWMSTileLayer,
    LLayerGroup, 
    LGeoJson,
    LCircleMarker,
    LTooltip,
    LControlLayers,
    appStationTooltip: StationTooltip,
    appMapLegend: MapLegend,
    appWaterQualityFloater: WaterQualityFloater
  },
  computed: {
    baseLayer() {
      return this.$store.state.baseLayer;
    },
    pointsLayer() {
      return this.$store.state.pointsLayer;
    },
    metricLayer() {
      return this.$store.state.metricLayer;
    },
    habitat() {
      return this.$store.state.habitat;
    },
    waterQuality() {
      return this.$store.state.waterQuality;
    },
    waterQualityGraph() {
      return this.$store.state.waterQualityGraph;
    },
    station() {
      return this.$store.state.station;
    },
    embayment() {
      return this.$store.state.embayment;
    },
    embStyle() {
      return () => {
        return {
          weight: 1,
          color: 'black',
          opacity: 0.5,
          fillColor: 'white',
          fillOpacity:0
        };
      };
    },
    regionStyle() {
      return () => {
        return {
          weight: 2,
          color: '#006400',
          fillColor: 'white',
          fillOpcacity: 0,
          interactive: false
        };
      };
    },
    nonCoastalStyle() {
      return () => {
        return {
          weight: 1,
          color: '#d3d3d3',
          fillColor: '#808080',
          fillOpacity: 0.5,
          interactive: false
        };
      };
    },
    smStyleCur() {
      return () => {
        return {
          stroke: false,
          fillColor: this.imageLibraryHabitat['salt marsh'].currentColor,
          fillOpacity: 0.85,
          interactive: false
        };
      };
    },	    
    smStyleHis() {
      return () => {
        return {
          stroke: false,
          fillColor: this.imageLibraryHabitat['salt marsh'].historicColor,
          fillOpacity: 0.5,
          interactive: false
        };
      };
    },
    egStyleCur() {
      return () => {
        return {
          stroke: false,
          fillColor: this.imageLibraryHabitat['eelgrass'].currentColor,
          fillOpacity: 0.85,
          interactive: false
        };
      };
    },	    
    egStyleHis() {
      return () => {
        return {
          stroke: false,
          fillColor: this.imageLibraryHabitat['eelgrass'].historicColor,
          fillOpacity: 0.5,
          interactive: false
        };
      };
    },
    tfStyleCur() {
      return () => {
        return {
          stroke: false,
          fillColor: this.imageLibraryHabitat['tidal flats'].currentColor,
          fillOpacity: 0.85,
          interactive: false
        };
      };
    },	    
    tfStyleHis() {
      return () => {
        return {
          stroke: false,
          fillColor: this.imageLibraryHabitat['tidal flats'].historicColor,
          fillOpacity: 0.5,
          interactive: false
        };
      };
    },
    metricStyleComp() {
      return () => {
        return {
          stroke: false,
          fillColor: '#FF0000',
          fillOpacity:0.5,
          interactive: false
        };
      };
    },
    optionsEmbayment() {
      return {
        onEachFeature: this.onEachEmbaymentFunction
      };
    },
    onEachEmbaymentFunction() {
      return (feature, layer) => {
        layer.on({
          click: (event) => {
            if (event.target.feature.properties.NAME == this.embayment) {
              this.stopFlyTo = true;
              this.$store.dispatch('setEmbayment', null);
              this.$refs.embaymentsBase.setOptionsStyle(this.embStyle);
              this.$nextTick(() => {
                this.stopFlyTo = false;
              })
            } else {
              this.$refs.ettMap.mapObject.flyToBounds(event.target.getBounds());
              this.$store.dispatch('setEmbayment', event.target.feature.properties.NAME);
              this.$refs.embaymentsBase.setOptionsStyle(this.embStyle);
              event.target.setStyle({weight: 2, color: 'red', opacity: 1});
              // this.$refs.embaymentsBase.mapObject.bringToFront();
              this.reorderLayers();
            }
          },
          mouseover: (event) => {
            let featureName = this.capitalizeFirstLetter(event.target.feature.properties.NAME)
            if ((this.metricLayer == true) && (feature.properties[this.habitat + '_percent_goal'] == '-999')) {
              event.target.getTooltip().setContent(featureName + '<br>Goal Not Yet Established');
            } else if (this.metricLayer == true) {
              event.target.getTooltip().setContent(featureName + '<br>% of 2050 Goal: ' + feature.properties[this.habitat + '_percent_goal'] + '%');
            } else {
              event.target.getTooltip().setContent(featureName);
            }    
          }
        });
        layer.bindTooltip('');
      };
    },
  },
  data () {
    return {
      imageLibraryHabitat: imageLibraryHabitat,
      basemaps: basemaps,
      basemapsWms: basemapsWms,
      zoom: 9,
      center: [41.7036, -70.8000],
      bkgrdGeojson: {
        tidalFlats: {
          name: 'tidal flats',
          data: {
            current: null,
            historic: null		    			
          }
        },
        saltMarsh: {
          name: 'salt marsh',
          data: {
            current: null,
            historic: null		    			
          }
        },
        eelGrass: {
          name: 'eelgrass',
          data: {
            current: null,
            historic: null		    			
          }
        }
      },
      staticGeojson: {
        snepRegion: {
          name: 'snep region',
          data: null
        },
        nonCoastal: {
          name: 'non-coastal',
          data: null
        }
      },
      embaymentsGeojson: null,
      stations: null,
      mapStyleObj: {
        height: Math.floor(window.innerHeight - 100) + 'px',
        width: '100%',
      },
      stopFlyTo: false
    };
  },
  watch: {
    '$store.state.embayment': {
      handler() {
        if ((this.embayment == null) && (this.stopFlyTo == false)) {
          this.$refs.embaymentsBase.setOptionsStyle(this.embStyle);  
          this.$refs.ettMap.mapObject.flyTo(this.center, this.zoom);     
        } else if (this.embayment == null) {
          this.$refs.embaymentsBase.setOptionsStyle(this.embStyle);          
        }
      }, immediate: true  
    },
    '$store.state.waterQuality': {
      handler() {
        this.$store.dispatch('setStation', null);
      }, immediate: true  
    }
  },
  methods: {
    circleColorer(parameterList) {
      if (parameterList.includes(this.waterQuality)) {
        return '#00B0F0';
      } else {
        return '#808080';
      }
    },
    activeCircleColorer(id, parameterList) {
      if (id == this.station) {
        return 'red';
      } else if (parameterList.includes(this.waterQuality)) {
        return '#00B0F0';
      } else {
        return '#808080';
      }
    },
    colorScale(value, geoHabitat) {
      if (geoHabitat == 'tidal flats') {
        return interpolatePurples(value);
      } else if (geoHabitat == 'salt marsh') {
        return interpolateOranges(value);
      } else if (geoHabitat == 'eelgrass') {
        return interpolateGreens(value)
      } else if (geoHabitat == 'diadromous fish') {
        return interpolateBlues(value);
      } else {
        return '#ffffff';
      }
    },
    embaymentColorer(value, geoHabitat) {
      if (value == -999) {
        return {
          stroke: false,
          weight: 1,
          color: 'black',
          opacity: 0.5,
          fillColor: '#adadad',
          fillOpacity: 0,
          interactive: false          
        }
      } else {
        let lowerBinValue = null;
        if (value < 20) {
          lowerBinValue = 0.0
        } else if (value >= 20 && value < 40) {
          lowerBinValue = 0.2;
        } else if (value >= 40 && value < 60) {
          lowerBinValue = 0.4;        
        } else if (value >= 60 && value < 80) {
          lowerBinValue = 0.6; 
        } else if (value >= 80 && value < 100) {
          lowerBinValue = 0.8; 
        } else if (value >= 100) {
          lowerBinValue = 1;
        }

        let featureColor = this.colorScale(lowerBinValue, geoHabitat);
        return {
          stroke: true,
          weight: 1,
          color: "black",
          opacity: 0.5,
          fillColor: featureColor,
          fillOpacity: 1.0,
          interactive: false
        };
      }
    },
    tidalFlatsColorer(feature) {
      return this.embaymentColorer(feature.properties['tidal flats_percent_goal'], 'tidal flats');
    },
    saltMarshColorer(feature) {
      return this.embaymentColorer(feature.properties['salt marsh_percent_goal'], 'salt marsh');
    },
    eelgrassColorer(feature) {
      return this.embaymentColorer(feature.properties['eelgrass_percent_goal'], 'eelgrass');
    },
    diadromousFishColorer(feature) {
      return this.embaymentColorer(feature.properties['diadromous fish_percent_goal'], 'diadromous fish');
    },
    parameterMapper(wqCounts) {
      const waterQualityMap = new Map();
      waterQualityMap.set('CHLA', 'chlorophyl-a');
      waterQualityMap.set('DO', 'dissolved oxygen');
      waterQualityMap.set('ECOLI', 'e. coli');
      waterQualityMap.set('ENT', 'enterococcus');
      waterQualityMap.set('PH', 'pH');
      waterQualityMap.set('SAL', 'salinity');
      waterQualityMap.set('TEMP', 'temperature');
      waterQualityMap.set('TN', 'nitrogen');
      waterQualityMap.set('TP', 'phosphorus');
      waterQualityMap.set('TURB', 'turbidity');    
      let activeParameters = [];
      Object.entries(wqCounts).forEach(([key, value]) => {
        if (value > 0 && key != 'CHLA') {
          activeParameters.push(waterQualityMap.get(key));
        } 
      });
      return activeParameters;

    },
    plotData(event, stationId, parameterList, stationEmbayment) {
      if (stationId == this.station) {
        this.$store.dispatch('offWaterQualityGraph');
        this.$store.dispatch('setStation', null);
      } else {
        this.$store.dispatch('onWaterQualityGraph');
        if (parameterList.includes(this.waterQuality)) {
          this.$nextTick(() => {
            this.$store.dispatch('onPlotWaterQualityGraph');
          })
        } else {
          this.$store.dispatch('offPlotWaterQualityGraph')
        }
        if (this.waterQuality == null) {
          this.$store.dispatch('setWaterQuality', 'nitrogen');
          this.$store.dispatch('setWaterQualityGraphVariable', 'nitrogen');
        } else {
          this.$store.dispatch('setWaterQualityGraphVariable', this.waterQuality);
        }
        this.$store.dispatch('setStation', stationId);
        this.$store.dispatch('setStationEmbayment', stationEmbayment);
      }
    },
    capitalizeFirstLetter(nameString) {
      let nameStringArray = nameString.split(' ');
      nameStringArray.forEach((word, index) => {
        if (word[0] == '(') {
          nameStringArray[index] = word.substring(0, 2) + word.slice(2).toLowerCase();
        } else {
          nameStringArray[index] = word[0] + word.slice(1).toLowerCase();
        }
      });
      return nameStringArray.join(' ');
    },
    onResize () {
      this.mapStyleObj = {
        height: Math.floor(window.innerHeight - 100) + 'px',
        width: '100%',
      }      
    },
    reorderLayers () {
      this.$nextTick(() => {
        if (('metric' in this.$refs) && (this.$refs.metric !== undefined)) {
          this.$refs.metric.mapObject.bringToFront();
        }
        if (('embaymentsBase' in this.$refs) && (this.$refs.embaymentsBase !== undefined)) {
          this.$refs.embaymentsBase.mapObject.bringToFront();
        }
      })
    }
  },
  created() {
    // fetch embayments
    fetch("./data/snep_subbasins_2024_wgs.json")
      .then(response => {
        return response.json()
      }).then(json => {
        let embaymentsGeoJson = json;

        embaymentsGeoJson.features.forEach(embayment => {
          embayment.properties['eelgrass_percent_goal'] = 0;
          embayment.properties['salt marsh_percent_goal'] = 0;
          embayment.properties['tidal flats_percent_goal'] = 0;
          embayment.properties['diadromous fish_percent_goal'] = 0;
        // fetch("./data/habitat_goals_percents.json")
        // .then(response => {
        //   return response.json()
        // }).then(json => {
        //   embaymentsGeoJson.features.forEach(embayment => {
        //     embayment.properties['eelgrass_percent_goal'] = json[embayment.properties.NAME].EG_PERCENT_GOAL;
        //     embayment.properties['salt marsh_percent_goal'] = json[embayment.properties.NAME].SM_PERCENT_GOAL;
        //     embayment.properties['tidal flats_percent_goal'] = json[embayment.properties.NAME].TF_PERCENT_GOAL;
        //     embayment.properties['diadromous fish_percent_goal'] = json[embayment.properties.NAME].DF_PERCENT_GOAL;
        //   })
          this.embaymentsGeojson = embaymentsGeoJson;
        })
    })
    // fetch habitats
    const fileArrayHabitat = {
      tidalFlats: {
        current: "./data/tidal_flats_current_wgs.geojson",
        historic: "./data/tidal_flats_historic_wgs.geojson"
      },
      saltMarsh: {
        current: "./data/salt_marsh_current_wgs.geojson",
        historic: "./data/salt_marsh_historic_wgs.geojson"
      },
      eelGrass: {
        current: "./data/eelgrass_current_wgs.geojson",
        historic: "./data/eelgrass_historic_wgs.geojson"
      },
    }
    Object.entries(this.bkgrdGeojson).forEach(([key, value]) => {
      Object.entries(value.data).forEach(([alt]) => {
        fetch(fileArrayHabitat[key][alt])
          .then(response => {
            return response.json()
          }).then(json => {
            this.bkgrdGeojson[key].data[alt] = json;
            // const year = json.name.split('_').pop()
            // this.$store.dispatch('setHabitatLegendYear', {habitat: value.name, period: alt, value: year});
        })					
      })
    })
    const fileArrayStatic = {
      snepRegion: "./data/snep_outer_boundary_wgs.json",
      nonCoastal: "./data/snep_non_coastal_2024_wgs.json"
    }
    Object.entries(this.staticGeojson).forEach(([key]) => {
      fetch(fileArrayStatic[key])
        .then(response => {
          return response.json()
        }).then(json => {
          this.staticGeojson[key].data = json;
      })
    })
    // fetch years file for legend
    const yearsFile = "./data/habitat_legend_years.json"
    fetch(yearsFile)
    .then(response => {
      return response.json()
    }).then(json => {
      json.forEach(habitat => {
        this.$store.dispatch('setHabitatLegendYear', {habitat: habitat.HABITAT, period: habitat.PERIOD, value: habitat.YEAR});
        // console.log(habitat.HABITAT);
      })
    })
    // fetch water quality stations
    fetch('./data/stations.json')
    .then(response => {
      return response.json()
    }).then(json => {
      this.stations = json.features;
      this.stations.forEach(station => {
        station.properties['parameter_list'] = this.parameterMapper(station.properties.wq_counts);
      });
    })
  },
  mounted() {
    window.addEventListener('resize', this.onResize);
  },
  updated() {
    this.reorderLayers();   
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.onResize) //stop memory leakage
  }
}
</script>
<style scoped>
.legend {
  position: absolute;
  top: 5%;
  right: 5%;
  z-index: 1000;
}
.floater-position {
  position: absolute;
  bottom: 5%;
  left: 5%;
  z-index: 1000; 
}
</style>