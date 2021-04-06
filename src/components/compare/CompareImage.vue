<template>
  <div class="container" ref="container">
    <div :class="(index === 0 ? 'left': 'right') + ' container__viewer'">
      <vl-map
        :load-tiles-while-animating="true"
        :load-tiles-while-interacting="true"
        data-projection="EPSG:3857"
        :class="(index === 0 ? 'left': 'right') + ' container__viewer__image'"
        ref="map"
      >
        <vl-view :zoom.sync="zoom" :center.sync="center" :rotation.sync="rotation" :max-zoom="maxZoom" :min-zoom="minZoom"></vl-view>

        <vl-layer-tile :extent="extent" @mounted="addOverviewMap" ref="baseLayer">
          <vl-source-zoomify
            :projection="projectionName"
            :urls="baseLayerURLs"
            :size="imageSize"
            :extent="extent"
            crossOrigin="Anonymous"
            ref="baseSource"
          />
        </vl-layer-tile>
      </vl-map>
    </div>
    <div class='buttonContainer'>
      <button class='submitCompare' v-on:click="sendWinnerLoser(index)" >
        Choose 
     </button>
    </div>
  </div>
</template>

<script>
import View from 'ol/View';
import OverviewMap from 'ol/control/OverviewMap';

import axios from 'axios';

export default {
  name: 'CytomineImage',
  props: {
    image: Object,
    index: Number,
    chosen: Boolean,
  },
  data() {
    return {
      zoom: null,
      minZoom: null,
      center: [0, 0],
      rotation: 0,
      baseSource: null,
      clientWidth: null,
      overview: null,
    };
  },
  computed: {
    projectionName() {
      return `CYTO-${this.image.id}`;
    },
    extent() {
      return [0, 0, this.image.width, this.image.height];
    },
    imageSize() {
      return [this.image.width, this.image.height];
    },
    imageServerURLs() {
      let urls = [
        'http://ims.awilson.website/image/tile?zoomify=' + this.image.fullPath,
        'http://ims2.awilson.website/image/tile?zoomify=' + this.image.fullPath
      ];
      return urls;
    },

    baseLayerURLs() {  
      let params = `&tileGroup={TileGroup}&x={x}&y={y}&z={z}&channels=0&layer=0&timeframe=0&mimeType=${this.image.mime}`;
      return this.imageServerURLs.map(url => url + params);
    },

    maxZoom() {
      return 21;
    },

  },
  methods: {
    setInitialZoom() {
      this.zoom = 16;
      this.minZoom = 15;
    },  
    setImageCenter() {
      this.center = [this.image.height / 2, this.image.height / 2];
    },
    async addOverviewMap() {

      await this.$refs.map.$createPromise; // wait for ol.Map to be created
      await this.$refs.baseLayer.$createPromise; // wait for ol.Layer to be created

      this.overview = new OverviewMap({
        view: new View({
          projection: 'EPSG:4326', minZoom: -9,
        }),
        layers: [this.$refs.baseLayer.$layer],
        tipLabel: 'overview map',
        target: this.$refs.overview,
        collapsible: false,
      });
      this.$refs.map.$map.addControl(this.overview);
    },

    
    async sendWinnerLoser(index) {
      if (index === 0) {
        let winnerId = this.$parent.imageIds[0];
        let loserId = this.$parent.imageIds[1];

        await axios.post('http://127.0.0.1:5000/props/elo?winner=' + winnerId + '&loser=' + loserId);
      } 
      else {
        let winnerId = this.$parent.imageIds[1];
        let loserId = this.$parent.imageIds[0];

        await axios.post('http://127.0.0.1:5000/props/elo?winner=' + winnerId + '&loser=' + loserId);
      }
      this.$parent.fetchNewImagesIds(); 
    }

  },
  mounted() {
    this.setInitialZoom();
    this.setImageCenter();
  },
};
</script>

<style scoped lang="scss">

.container {
  display: flex;
  background: #f7f3ff;
  height: 100%;
  flex-flow: column;
  width: 100%;

  .container__viewer {
    display: flex;
    height: calc(100% - 5rem);
    background: #f7f3ff;

    > .container__viewer__image {
      width: 100%;
      overflow: hidden;
      height: 100%;
    }

    > .left {
      border-right: 1px solid #2c3e506b;
    }

    > .right {
      border-left: 1px solid #2c3e506b;
    }
  }

  > .container__panels {
    margin-top: -4rem;
    z-index: 1;
    text-align: right;
    margin-right: 1rem;

    > .custom-overview {
      position: relative;
      left: 1rem;

    }
  }

  > .chosen {
    background: #b395f3;
  }
}

.submitCompare {
    width: 90%;
    align-self: center;
    min-height: 40px;
    font-size: 24px;
    margin-top: 5px;
    margin-bottom: 5px;
}

.buttonContainer {
  display: flex;
  background: #f7f3ff;
  height: 100%;
  flex-flow: column;
  width: 100%;
  border: 2px solid black;
  margin: 2px;
  padding: 2px;

}
</style>

<style lang="scss">
  .custom-overview {
    > .ol-overviewmap:not(.ol-collapsed) {
      backdrop-filter: blur(5px);
      background: rgba(255, 255, 255, .4);
    }
    > .ol-overviewmap .ol-overviewmap-map {
      border: none;
    }
  }
</style>
