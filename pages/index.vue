<template lang="pug">
  .container
    b-form-file(
      id="file-large",
      size="lg",
      placeholder="Выберете файл или переместите его сюда...",
      @change="selectedFile",
      browse-text="Выбрать файл",
      :style="{marginBottom: '30px'}"
    )

    b-button(v-if="resized", size="lg", @click.prevent="download", variant="success") Скачать

    resize-form(v-if="uploadFileId", :upload-file-id="uploadFileId", @resize="onResize")

    b-modal(ref="modal-1")
      p.my-4 Ресайзинг еще не закончен, подождите и попытайтесь скачать снова

    b-modal(ref="modal-2")
      p.my-4 Что-то пошло не так
</template>

<script>
import ResizeForm from '~/components/ResizeForm';

export default {
  name: 'IndexPage',
  components: { ResizeForm },
  data(){
    return {
      uploadFileId: '',
      resized: false,
      file: null,
    }
  },
  methods:{
    selectedFile(event){
      console.log(event.target.files)
      this.upload(event.target.files)
    },
    async upload(files){
      files = this.fileListToArray(files)
      const file = files[0]
      try{
        this.uploadFileId = await this.uploadFile({
          file
        })
      } catch (e){
        console.warn(e)
      }
    },
    async uploadFile(file){
      const request = new FormData()
      for (const [prop, value] of Object.entries(file)) {
        request.append(prop, value)
      }
      return await this.$axios.post('/files/', request)
    },
    fileListToArray(files) {
      const result = []
      for (const file of files) {
        result.push(file)
      }
      return result
    },
    async download(){
      try {
        await this.downloadFile()
      } catch (e){
        console.warn(e)
        if(e.response.status === 423){
          this.$refs['modal-1'].show()
        }else {
          this.$refs['modal-2'].show()
        }
      }

    },
    async downloadFile() {
      try {
        const response = await this.$axios.get(`/files/${this.uploadFileId}/view`)
        const blob = response.blob()
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.href = url
        a.target = '_blank'
        a.download = name
        document.body.appendChild(a)
        a.click()
        document.body.removeChild(a)
        URL.revokeObjectURL(url)
      } catch (e) {
        console.warn(e)
        throw e
      }
    },
    onResize(event){
      this.resized = true
    },
  }
}
</script>

<style lang="stylus">
.container
  display flex
  flex-direction column
  align-items center
  padding-top 15vh
.dropdown-toggle::after
  position absolute
  right 20px
  height 10px
  top 17px
</style>
