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
              v-text-field(
                label='タイトル*',
                required,
                v-model='title',
                :disabled='loadFlag'
              )
            v-col(cols='12')
              v-text-field(
                label='お困り事情報*',
                required,
                v-model='description',
                :disabled='loadFlag'
              )
            v-col(cols='12')
              v-select(
                :items='item',
                label='タグ',
                v-model='tag',
                :disabled='loadFlag'
              )
            v-file-input(cols='12', v-model='image', :disabled='loadFlag')
        small *indicates required field
      v-card-actions
        v-spacer
        v-btn(
          color='blue darken-1',
          text,
          @click='closeModal',
          :disabled='loadFlag'
        )
          | Close
        v-btn(
          color='blue darken-1',
          text,
          @click='getPosition',
          :disabled='loadFlag'
        )
          | Send
  v-dialog(v-model='errorDialog', persistent, width='360')
    v-card(align='center')
      v-card-title.justify-center
        | 最初からやり直してください
      v-card-text
        | {{ errorText }}
      v-btn.mb-3(@click='errorDialogClose', outlined, color='red lighten-2') OK
  v-dialog(v-model='successDialog', persistent, width='360')
    v-card(align='center')
      v-card-title.justify-center
        | 成功
      v-card-text
        | 投稿しました
      v-btn.mb-3(
        @click='successDialog = false',
        outlined,
        color='green lighten-2'
      ) OK
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'nuxt-property-decorator'
import { mdiBorderColor } from '@mdi/js'

@Component({})
export default class Modal extends Vue {
  @Prop({ type: String, required: true })
  basicAuth: String | undefined

  @Prop({ type: String, required: true })
  appUrl: String | undefined

  svgPath = mdiBorderColor
  dialog: boolean = false
  image = []
  description: string | Blob = ''
  tag: string | Blob = ''
  title: string | Blob = ''
  errorDialog = false
  errorText = ''
  successDialog = false
  loadFlag = false
  item = ['視覚', '聴覚', '車椅子', '松葉杖']

  closeModal() {
    this.title = ''
    this.image = []
    this.description = ''
    this.tag = ''
    this.errorText = ''
    this.dialog = false
  }

  getPosition() {
    this.loadFlag = true
    this.successDialog = true
    this.closeModal()
    // navigator.geolocation.getCurrentPosition(
    //   this.sendData,
    //   () => this.errorFunc('位置情報が取得できませんでした'),
    //   {
    //     enableHighAccuracy: true,
    //   },
    // )
  }

  errorFunc(text: string) {
    this.errorText = text
    this.errorDialog = true
    this.loadFlag = false
  }

  errorDialogClose() {
    this.errorDialog = false
  }

  sendData(data: GeolocationPosition) {
    if (this.title === '') this.errorFunc('タイトルを入力してください')
    if (this.description === '') this.errorFunc('説明を入力してください')

    const p = data.coords

    const formData = {
      lat: p.latitude,
      lon: p.longitude,
      title: this.title,
      message: this.description,
      user_id: 1,
      handicap_id: this.item.findIndex((e) => e === this.tag),
    }

    const config = {
      header: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${this.basicAuth}`,
      },
    }

    if (this.appUrl) {
      console.log(this.appUrl, formData)
      this.$axios
        .post(this.appUrl as string, { formData, config })
        .then(() => {
          this.successDialog = true
          this.closeModal()
        })
        .catch(() => {
          this.errorFunc('送信できませんでした')
        })
    } else {
      this.errorFunc('送信できませんでした')
    }
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
