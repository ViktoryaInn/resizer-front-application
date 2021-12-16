<template lang="pug">
  div(:style="{ width: 'fit-content' }")
    b-spinner(v-if="loading", type="grow" label="Spinning")
    .resize-form(:style="{ filter: loading ? 'blur(2px)' : 'unset' }")
      .resize-form__item
        b-form-group(label="Выберите тип трансформации", :description="resizeModel.m ? descriptionMap.get(resizeModel.m) : ''")
          b-dropdown.m-md-2(id="dropdown-1", :text="resizeModel.m ? localeMap.get(resizeModel.m) : ''", :style="{ width: '470px', height: '38px' }", variant="outline-primary")
            b-dropdown-item(
              v-for="(item, index) in TransformationStrategies",
              :key="index",
              @click.prevent="selectTransformationStrategy(item)"
            ) {{localeMap.get(item)}}

      .resize-form__item
        b-form-group(label="Введите ширину")
          b-form-input(type="number", v-model="resizeModel.w")
        b-form-group(label="Введите высоту")
          b-form-input(type="number", v-model="resizeModel.h")

      b-button(variant="primary", :style="{ width: '200px', alignSelf: 'center' }", @click.prevent="doResize()") Начать ресайз

    div
      b-modal(ref="modal-1")
        p.my-4 Необходимо заполнить все поля, для ресайза изображения

      b-modal(ref="modal-2")
        p.my-4 Что-то пошло не так
</template>

<script>
const TransformationStrategies = ['cover', 'contain', 'fit']
const localeMap = new Map([
  ['cover', 'Заполнить по наименьшей стороне'],
  ['contain', 'Вписать по наибольшей стороне'],
  ['fit', 'Вписать в размер']
])

const descriptionMap = new Map([
  ['cover', 'Пропорции будут сохранены'],
  ['contain', 'Пропорции будут сохранены'],
  ['fit', 'Пропорции не будут сохранены']
])

export default {
  name: 'ResizeForm',
  props: {
    uploadFileId: {
      type: String,
      default() {
        return ''
      },
    },
  },
  data(){
    return {
      resizeModel: {
        w: 0,
        h: 0,
        m: '',
      },
      TransformationStrategies: Object.freeze(TransformationStrategies),
      localeMap: Object.freeze(localeMap),
      descriptionMap: Object.freeze(descriptionMap),
      loading: false,
    }
  },
  watch:{
    uploadFileId(){
      this.resizeModel.w = 0
      this.resizeModel.h = 0
      this.resizeModel.m = ''
    }
  },
  methods:{
    selectTransformationStrategy(value){
      this.resizeModel.m = value
    },
    async doResize() {
      if(!this.resizeModel.w || !this.resizeModel.h || !this.resizeModel.m) {
        this.$refs['modal-1'].show()
        return false
      }

      try {
        this.loading = true
        const result = await this.$axios.post(`files/${this.uploadFileId}/resize`, { params: this.resizeModel })
        this.$emit('resize', result)
        this.loading = false
      } catch (e){
        console.warn(e)
        this.loading = false
        this.$refs['modal-2'].show()
      }
    },
  },
}
</script>

<style lang="stylus" scoped>
.spinner-grow
  position relative
  top 150px
  left 255px

.dropdown-toggle::after
  position absolute
  right 20px
  height 10px
  top 17px

.resize-form
  width fit-content
  padding 30px
  display flex
  flex-direction column
  justify-content center
  gap 15px
  &__item
    display flex
    align-items center
    justify-content space-between
</style>
