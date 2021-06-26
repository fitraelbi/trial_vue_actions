<template>
  <div id="app">
    <vl-map @click="clickCoordinate = $event.coordinate" :load-tiles-while-animating="true" :load-tiles-while-interacting="true" :data-projection="epsg" style="height: 400px">
      <vl-view :zoom.sync="zoom" :center.sync="center" :rotation.sync="rotation"></vl-view>

      <vl-layer-tile>
        <vl-source-osm></vl-source-osm>
      </vl-layer-tile>

      <vl-layer-vector>
        <vl-source-vector :features.sync="features"></vl-source-vector>
      </vl-layer-vector>

       <vl-interaction-select :features.sync="selectedFeatures" v-if="drawType == null">
        <template slot-scope="select">
          <!-- select styles -->
          <vl-style-box>
            <vl-style-stroke color="#423e9e" :width="7"></vl-style-stroke>
            <vl-style-fill :color="[254, 178, 76, 0.7]"></vl-style-fill>
            <vl-style-circle :radius="5">
              <vl-style-stroke color="#423e9e" :width="7"></vl-style-stroke>
              <vl-style-fill :color="[254, 178, 76, 0.7]"></vl-style-fill>
            </vl-style-circle>
          </vl-style-box>
          <vl-style-box :z-index="1">
            <vl-style-stroke color="#d43f45" :width="2"></vl-style-stroke>
            <vl-style-circle :radius="5">
              <vl-style-stroke color="#d43f45" :width="2"></vl-style-stroke>
            </vl-style-circle>
          </vl-style-box>
          <!--// select styles -->

          <!-- selected feature popup -->
          <vl-overlay class="feature-popup" v-for="feature in select.features" :key="feature.id" :id="feature.id"
                      :position="pointOnSurface(feature.geometry)" :auto-pan="true" :auto-pan-animation="{ duration: 300 }">
            <template >
              <section class="card">
                <header class="card-header">
                  <p class="card-header-title">
                    Feature ID {{ feature.id }}
                  </p>
                  <a class="card-header-icon" title="Close"
                     @click="selectedFeatures = selectedFeatures.filter(f => f.id !== feature.id)">
                    <b-icon icon="close"></b-icon>
                  </a>
                </header>
                <div class="card-content">
                  <div style="padding: 10px;" class="content">
                    <p>
                      Note : {{ feature.properties.note}}
                    </p>
                    <p>
                      Latitude :  {{ feature.properties.lat}}
                    </p>
                    <p>
                      Longitude :  {{ feature.properties.lon}}
                    </p>
                    <p>
                      Utm Code :  {{ feature.properties.utm_code}}
                    </p>
                    <p>
                      Utm SRID :  {{ feature.properties.utm_srid}}
                    </p>
                    <div class="row d-flex justify-content-center">
                      <button
          type="button"
          class="btn col-4 btn-modal project-create"
          @click="sendProject"
        >
          Update
        </button>
                        <button
          type="button"
          class="btn col-4 btn-modal project-close"
          @click="hideModal"
        >
          Delete
        </button>
        
                    </div>
                  </div>
                </div>
              </section>
            </template>
          </vl-overlay>
          <!--// selected popup -->
        </template>
      </vl-interaction-select>
    </vl-map>
    <p v-if="loading">
      Loading features, please wait...
    </p>
    <p v-if="features.length > 0">
      Loaded features: {{ features.map(feature => feature.id) }}
    </p>
  </div>
</template>

<script>
  import { findPointOnSurface } from 'vuelayers/lib/ol-ext'
  import axios from 'axios'
  export default {
    data () {
      return { 
        zoom: 18,
        center: [106.816666, 	-6.200000],
        rotation: 0,
        features: [],
        loading: false,
        epsg: 'EPSG:4326',
        data_api: [],
        clickCoordinate: undefined,
        selectedFeatures: [],
        drawType: undefined,
      }
    },
    mounted () {
      const username = 'admin'
      const password = 'admin'
      const token = Buffer.from(`${username}:${password}`, 'utf8').toString('base64')
      this.loading = true
      this.loadFeatures().then(features => {
        this.features = features.map(Object.freeze)
        this.loading = false
      })
      axios.get('https://backend.widesight.id/image_objects/?as_geojson=true', {
  headers: {
 'Authorization': `Basic ${token}`
},
}).then(res => {
  console.log((res.data.features[0]).geometry.coordinates)
  this.center = (res.data.features[0]).geometry.coordinates
  this.data_api = res.data.features
  })
    },
    methods: {
      // emulates external source
      pointOnSurface: findPointOnSurface,
      loadFeatures () {
        return new Promise(resolve => {
          setTimeout(() => {
            // generate GeoJSON random features
            resolve(this.data_api)
          }, 3000)
        })
      },

    },
  }
</script>

<style scoped>
.project-close {
  color: white;
  background: #ea868f;
  margin: 10px;
}
.project-close:hover {
  background: #f1aeb5;
  color: white;
}
.project-create {
  color: white;
  background: #8cc152;
  margin: 10px;
}
.project-create:hover {
  background: #a0D468;
  color: white;
}
</style>