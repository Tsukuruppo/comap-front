<template lang="pug">
.wrapper
  #map.map
  v-dialog(v-model='dialog', persistent, max-width='600px')
    template(v-slot:activator='{ on, attrs }')
      v-btn(color='primary', dark, v-bind='attrs', v-on='on')
        | Open Dialog
    v-card
      v-card-title
        span.text-h5 投稿フォーム
      v-card-text
        v-container
          v-row
            v-col(cols='12')
              v-text-field(label='タイトル*', required, v-model='title')
            v-col(cols='12')
              v-text-field(label='お困り事情報*', required, v-model='description')
            v-col(cols='12')
              v-autocomplete(
                :items='[0, 1, 2, 3, 4, 5]',
                label='タグ',
                v-model='tag',
                multiple
              )
            v-file-input(cols='12', v-model='image')
        small *indicates required field
      v-card-actions
        v-spacer
        v-btn(color='blue darken-1', text, @click='closeModal')
          | Close
        v-btn(color='blue darken-1', text, @click='getPosition')
          | Send
  v-dialog(v-model='errorDialog', persistent, width='360')
    v-card(align='center')
      v-card-title.justify-center
        | 最初からやり直してください
      v-card-text
        | {{ errorText }}
      v-btn.mb-3(@click='errorDialogClose', outlined, color='red lighten-2') OK
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
import mapboxgl from 'mapbox-gl'
import MapboxLanguage from '@mapbox/mapbox-gl-language'
import 'mapbox-gl/dist/mapbox-gl.css'

@Component({
  asyncData({ $config }) {
    const accessToken = $config.accessToken
    return { accessToken }
  },
})
export default class Index extends Vue {
  map!: mapboxgl.Map
  accessToken: string = ''
  dialog: boolean = false
  image: string | Blob = ''
  description: string | Blob = ''
  tag: string | Blob = ''
  title: string | Blob = ''
  errorDialog = false
  errorText = ''

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
    )

    this.map.addControl(new mapboxgl.NavigationControl(), 'bottom-right')
  }

  closeModal() {
    this.title = ''
    this.image = ''
    this.description = ''
    this.tag = ''
    this.errorText = ''
    this.dialog = false
  }

  getPosition() {
    navigator.geolocation.getCurrentPosition(
      this.sendData,
      () => this.errorFunc('位置情報が取得できませんでした'),
      {
        enableHighAccuracy: true,
      },
    )
  }

  errorFunc(text: string) {
    this.errorText = text
    this.errorDialog = true
  }

  errorDialogClose() {
    this.errorDialog = false
  }

  sendData(data: GeolocationPosition) {
    if (this.title === '') this.errorFunc('タイトルを入力してください')
    if (this.description === '') this.errorFunc('説明を入力してください')

    const p = data.coords
    const formData = new FormData()
    formData.append('x', '' + p.latitude)
    formData.append('y', '' + p.longitude)
    formData.append('id', '0')
    formData.append('user_id', '0')
    formData.append('image', this.image)
    formData.append('title', this.title)
    formData.append('description', this.description)
    formData.append('tag', this.tag)

    const config = {
      header: {
        'Content-Type': 'multipart/form-data',
      },
    }

    this.$axios
      .post('url', { formData, config })
      .then(() => {
        this.closeModal()
      })
      .catch(() => {
        this.errorFunc('送信できませんでした')
      })
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

.wrapper {
  position: relative;
}

.button {
  position: absolute;
  top: 0;
  left: 0;
}
</style>
