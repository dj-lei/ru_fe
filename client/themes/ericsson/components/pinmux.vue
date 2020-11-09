<template lang="pug">
  div
    v-flex.page-col-content(xs12, lg9, xl10)
      .headline.animated.fadeInUp {{ title }}
      v-row(justify="space-between")
        v-col(class="d-flex", cols="12", sm="6")
          v-select(v-model="asic_select", :items="asic", label="asic", dense, outlined)
        v-text-field(v-model="revision", :counter="10", label="revision", required)
      //- v-card(
      //-   class="mx-auto mt-6"
      //- )
      v-progress-linear(
        v-if='is_show'
        ref='progress_linear'
        indeterminate=false
        absolute
        color="cyan"
      )
      input(
            ref='browseFile'
            type='file'
            style='display:none'
            @change='upload'
            )
      v-btn.mt-5.animated.fadeInUp.wait-p2s(ref='upload', color='secondary', elevation="2", @click='clickInputFile', outlined, block, x-large)
        span UPLOAD
      v-btn.mt-5.animated.fadeInUp.wait-p2s(ref='download', color='secondary', elevation="2", @click='download', outlined, block, disabled, x-large)
        span DOWNLOAD
      v-spacer
      .subtitle-1.mt-3 {{ error_info }}
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      title: 'PINMUX',
      asic: ['xenon', 'radon'],
      fileName: 'None',
      is_show: false,
      revision: '',
      asic_select: '',
      error_info: ''
    }
  },
  methods: {
    clickInputFile () {
      this.$refs.download.disabled = true
      this.error_info = ''
      // this.is_show = true
      this.$refs.browseFile.click()
    },
    upload (e) {
      let formData = new FormData()
      formData.append("file", e.target.files[0])
      formData.append("asic", this.asic_select)
      formData.append("revision", this.revision)
      let config = {
        headers: {
        'Content-Type': 'multipart/form-data'
        }
      }
      // this.$refs.progress_linear.indeterminate = true
      axios.post("http://10.166.152.49/es/upload_pinmux/", formData, config).then(
        (response)=>{
          this.fileName = response.data
          // this.$refs.progress_linear.indeterminate = false
          this.$refs.download.disabled = false
      }, (error) => {
        this.error_info = 'ERROR:asic and revision must fill in or the format of the file is not correct or Pinmux page is not the first sheet!'
      })
      this.error_info = 'INFO:Upload Success!'
    },
    download () {
      axios.get("http://10.166.152.49/es/download_pinmux/", {
        params: {
          fileName: this.fileName,
        },
        responseType: 'blob' ,
      })
      .then(response => {
        const blob = new Blob([response.data], { type: 'application/txt' })
        const link = document.createElement('a')
        link.href = URL.createObjectURL(blob)
        link.download = this.fileName
        link.click()
        URL.revokeObjectURL(link.href)
      })
    }
  }
}
</script>
