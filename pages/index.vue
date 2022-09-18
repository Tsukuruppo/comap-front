<template lang="pug">
.wrapper
  #map.map
  Modal(:basicAuth='basicAuth', :appUrl='appUrl')
  InfoModal
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
import mapboxgl from 'mapbox-gl'
import MapboxLanguage from '@mapbox/mapbox-gl-language'
import 'mapbox-gl/dist/mapbox-gl.css'

@Component({
  asyncData({ $config }) {
    const accessToken = $config.accessToken
    const basicAuth = $config.basicAuth
    const appUrl = $config.appUrl
    return { accessToken, basicAuth, appUrl }
  },
})
export default class Index extends Vue {
  map!: mapboxgl.Map
  accessToken: string = ''

  mounted() {
    this.map = new mapboxgl.Map({
      accessToken: this.accessToken,
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [139.7671, 35.6812],
      zoom: 9,
    })

    this.map.addControl(
      new MapboxLanguage({
        defaultLanguage: 'ja',
      }),
    )

    this.map.addControl(
      new mapboxgl.GeolocateControl({
        positionOptions: {
          enableHighAccuracy: true,
        },
        trackUserLocation: true,
        showUserHeading: true,
      }),
      'bottom-right',
    )

    this.map.addControl(new mapboxgl.NavigationControl(), 'bottom-right')
  }
}
</script>

<style lang="scss" scoped>
.map {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
}
</style>
