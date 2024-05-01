<template>
  <div>
    <div class="row">
      <app-habitat-graph-header style="width: 100%"></app-habitat-graph-header>
    </div>
    <div class="row">
      <app-habitat-graph style="width: 100%"></app-habitat-graph> 
    </div>
    <br>
    <br>
    <br>
    <br>
    <br>
    <div class="row" style="height: 100px">
      <div class="col-3 align-self-center" style="margin-right: 0px; padding-right: 0px">
        <a href="https://www.epa.gov/waterdata/water-quality-data" target="_blank">
          <img src="../assets/logos/wqX-logo_clip.gif" style="width: 100%" alt="EPA WQX logo">
        </a>
      </div>
      <div class="col-9 align-self-center">
        <span>
          <b>Water quality data is based on  data acquired from 
            <a href="https://www.epa.gov/waterdata/water-quality-data" target="_blank">
              WQX
            </a>
            as of __date__.</b>
        </span>
      </div> 
    </div>
  </div>
</template>

<script>
import HabitatGraph from './subs/HabitatGraph.vue'
import HabitatGraphHeader from './subs/HabitatGraphHeader.vue'
import { habitatSelections } from '../lib/constants'
import { waterQualitySelections } from '../lib/constants'
export default {
  data () {
    return {
        habitatSelections: habitatSelections,
        waterQualitySelections: waterQualitySelections,
        activeTab: null,
        disableHabitatTab: null,
        disableWaterQualityTab: null
    }
  },
  computed: {
    habitatCapital() {
      const titles = this.$store.state.habitat.split(" ");
      const capitalTitle = titles.map((word) => {
        return word[0].toUpperCase() + word.substring(1);
      }).join(" ");
      return capitalTitle;
    },
    habitatIndexLayer() {
      return this.$store.state.habitatIndexLayer;
    },
    blockHabitatGraph() {
      return this.$store.state.blockHabitatGraph;
    }
  },
  components: {
    appHabitatGraph: HabitatGraph,
    appHabitatGraphHeader: HabitatGraphHeader
  }
}
</script>

<style scoped>
.bottom-row {
  width: 100%;
  height: 100px;
}

.bottom-row img {
  margin: auto;
  vertical-align: middle; 
  display: inline-block;
}
</style>
