<template>
  <div id="app">
    <div class="g-wraper">
      <div class="m-part">
        <mapDrag @drag="dragMap" class="mapbox"></mapDrag>
      </div>
    </div>
  </div>
</template>

<script>
import mapDrag from './components/mapDrag'
export default {
  name: 'app',
  components: {
    mapDrag
  },
  data () {
    return {
      dragData: {
        lng: null,
        lat: null
      }
    }
  },
  methods: {
    dragMap (data) {
      this.dragData = {
        lng: data.position.lng,
        lat: data.position.lat
      }
    }
  }
}
</script>

<style>
body{ margin: 0; }
.g-wraper{ width: 1000px; margin: 0 auto; color: #666; font-size: 16px; line-height: 30px; }
.m-part{ margin-bottom: 30px; }
.m-part .mapbox{ width: 600px; height: 400px; margin-bottom: 20px; float: left; }
.amap-ui-misc-positionpicker{display: none;}
.amap-logo{display: none;}
.amap-copyright{display: none !important;}
</style>
