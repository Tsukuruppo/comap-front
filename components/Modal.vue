<template lang="pug">
div
  v-dialog(v-model='dialog', persistent, max-width='600px')
    template(v-slot:activator='{ on, attrs }')
      v-icon.icon(color='primary', dark, v-bind='attrs', v-on='on')
        | {{ svgPath }}
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
import { mdiBorderColor } from '@mdi/js'

@Component({})
export default class Modal extends Vue {
  svgPath = mdiBorderColor
  dialog: boolean = false
  image = []
  description: string | Blob = ''
  tag: string | Blob = ''
  title: string | Blob = ''
  errorDialog = false
  errorText = ''

  closeModal() {
    this.title = ''
    this.image = []
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
    formData.append('image', this.image[0])
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
.icon {
  position: absolute;
  top: 8px;
  left: 8px;
}
</style>
