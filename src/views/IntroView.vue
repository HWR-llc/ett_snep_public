<template>
  <div class="container-fluid" >
    <div class="row spacer"></div>
    <div class="row header-footer">
    </div>
    <div class="row" style="min-height: 500px">
      <div class="col-3 left-text">
        <div class="row">
          <div class="col-12 text-center" style="background-color: #CEE1A0">
            <h4>
              Explore <br> Coastal Habitats and Water Quality in Southeast New England
            </h4>
          </div>
        </div>
        <div class="row" style="padding: 30px; background-color: #FFFFFF">
          <p>
          The <b>Ecohealth Tracking Tool</b> is a gateway for exploring coastal habitats and water quality for the EPA Southeast New England Program (SNEP) region... or take a closer look data for your favorite beach, salt marsh, or estuary.
          </p>
          <p>
            This took was developed based on the <a href="https://massbaysecohealth.org/#/" target="_blank">MassBays Ecohealth Tracking Tool</a> which displays data for the Massachusetts Bay region.
          </p>
        </div>
        <div class="row justify-content-center">
          <img src="../assets/logos/epa_logo_blue1.png" class="left-logo" alt="EPA logo">
        </div>
          <br>
        <div class="row justify-content-center">
          <img src="../assets/logos/SNEP logo.png" class="left-logo" alt="EPA-SNEP logo">
        </div>
        <br><br>
      </div>
      <div class="col-9 right-image">
        <br><br><br><br><br>
        <div class="row justify-content-center">
          <img src="../assets/logos/ett_snep_square_no_border.svg" class="right-logo" alt="EPA-SNEP logo">
        </div>
        <br><br class="d-none d-md-block">
        <div class="d-none d-md-block">
          <div class="row justify-content-center">
            <b-button class="water-quality-background no-border" @click="()=>$router.push('explorer')" size="lg"><b>Start Exploring</b></b-button>
          </div>
        </div>
        <div class="row justify-content-center d-md-none">
          <b-alert show variant="warning">Mobile Not Supported. Please come back on a laptop or desktop computer.</b-alert>
        </div>        
        <br class="d-none d-md-block">
      </div>
    </div>
    <div class="row header-footer">
    </div>
  </div>
</template>

<script>
export default {
  name: 'IntroView',
  data () {
    return {
      imageWidth: null,
      textWidth: null,
      iconWidth: null,
      textAlign: null

    }
  },
  methods: {
    onResize () {
      if (window.innerWidth < 992) {
        this.imageWidth = 90;
        this.textWidth = 90;
        this.iconWidth = 75;
        this.textAlign = null;
      } else {
        this.imageWidth = 35;
        this.textWidth = 60;
        this.iconWidth = 15;
        this.textAlign = 'center';
      }
    }
  },
  mounted() {
    window.addEventListener('resize', this.onResize);
    this.$nextTick(() => {
      this.onResize();
    });
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.onResize); // stop memory leaks;
  }
}

</script>

<style scoped>
.bottom-logo {
  line-height: 150px;
  width: 100%;
}

.left-logo {
  width: 30%;
  min-width: 110px;
}
.spacer {
  height: 50px;
  background-color: #FFFFFF;
}
.header-footer {
  height: 15px;
  background-color: #0095D6;
}
.left-text {
  color: #0B5395;
}
.right-image {
  background-image: url("../assets/imgs/landing_page.png");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
}
.right-logo {
  width: 20%;
  background-color: #FFFFFF;
}
</style>
