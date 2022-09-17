<template lang="pug">
div
  v-icon.icon(role='img', @click='dialogOpen') {{ svgPath }}
  v-dialog(v-model='dialog', persistent, max-width='600px')
    v-card(align='center')
      v-card-title.justify-center
        | お困りごと投稿サービスについて
      v-card-text(align='left')
        | 地図情報と結び付いた、リアルなお困りごとを発信することで、そのスポットを訪れた人が、あなたのお困りごと情報を読み、疑似体験することができます。<br/>
        | 使い方は簡単です。左上の投稿フォームから、お困りごと情報を記載して送信するだけ。<br/>
        | タイトルやタグ、画像をつけるとより多くの人に伝わりやすくなります。<br/>
        | 投稿した内容は、COMAPアプリと連携し、マップに表示されます。
      v-btn.mb-3(@click='dialogClose', outlined, color='red lighten-2') OK
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
import { mdiInformation } from '@mdi/js'

@Component({
  asyncData({ $config }) {
    console.log('tokennnfakfkadhfkjanfkladfnglka')
    const accessToken = $config.accessToken
    return { accessToken }
  },
})
export default class InfoModal extends Vue {
  svgPath = mdiInformation
  dialog: boolean = false

  dialogOpen() {
    this.dialog = true
  }

  dialogClose() {
    this.dialog = false
  }
}
</script>

<style lang="scss" scoped>
.icon {
  position: absolute;
  top: 8px;
  right: 8px;
  color: black;
}
</style>
