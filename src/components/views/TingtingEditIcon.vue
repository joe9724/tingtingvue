<template>
  <div>
    <h5 class="text-center">编辑Icon</h5>
    <section class="content">
      <div class="row">
        <div class="col-md-12">
          <el-form ref="form" :model="form" label-width="80px">
            <el-form-item label="名称">
              <el-input v-model="form.name"></el-input>
            </el-form-item>
            <el-form-item label="类别">
              <el-input v-model="form.type"></el-input>
            </el-form-item>
            <el-form-item label="类别ID">
              <el-input v-model="form.targetId"></el-input>
            </el-form-item>
            <el-form-item label="链接">
              <el-input v-model="form.webUrl"></el-input>
            </el-form-item>
            <el-form-item label="图片">
              <img v-bind:src="imgUrl"/>
              <vue-core-image-upload
                class="btn btn-primary"
                :crop="false"
                @imageuploaded="imageUploaded"
                :data="data"
                :max-file-size="5242880"
                inputOfFile="file"
                text="选择图片"
                :credentials="false"
                :url="uploadUrl">

              </vue-core-image-upload>
            </el-form-item>
            <el-form-item label="状态">
              <el-radio-group v-model="form.status">
                <el-radio label="正常"></el-radio>
                <el-radio label="锁定"></el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item>
              <!--<vue-editor id="editor"
                          useCustomImageHandler
                          @imageAdded="handleImageAdded" v-model="htmlForEditor">
              </vue-editor>-->
              <el-button type="primary" @click="onSubmit">确定</el-button>
              <!--<el-button>确定</el-button>-->
            </el-form-item>
          </el-form>
        </div>
      </div>
    </section>
  </div>
</template>
<script>
  import configParams from '../../config'
  import api from '../../api'
  import { VueEditor } from 'vue2-editor'
  import VueCoreImageUpload from '../../../node_modules/vue-core-image-upload/src/vue-core-image-upload.vue'

  export default {
    components: {
      VueEditor,
      'vue-core-image-upload': VueCoreImageUpload
    },
    data () {
      return {
        src: 'http://img1.vued.vanthink.cn/vued0a233185b6027244f9d43e653227439a.png',
        htmlForEditor: '',
        uploadUrl: '',
        imgUrl: '',
        form: {
          name: '',
          region: '',
          date1: '',
          date2: '',
          delivery: false,
          resource: '',
          desc: '',
          status: '',
          author: '',
          content: '',
          type: '',
          targetId: '',
          webUrl: '',
          fileList2: [{
            name: 'food.jpeg',
            url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'
          }]
        }
      }
    },
    created () {
      this.uploadUrl = configParams.uploadURI
      var iconId = '0'
      if (this.$route.query.iconId) {
        iconId = this.$route.query.iconId
        this.iconId = this.$route.query.iconId
      }
      api.request('get', 'icon/detail?userid=1&iconId=' + iconId)
        .then(response => {
          var data = response.data.body.data
          // this.form.pass = '******'
          this.form.name = data.name
          this.form.type = data.type
          this.form.targetId = data.jumpid
          this.form.webUrl = data.webUrl
          this.imgUrl = data.cover
          if (data.status === 0) {
            this.form.status = '正常'
          } else {
            this.form.status = '锁定'
          }
        })
        .catch(error => {
          // this.$store.commit('TOGGLE_LOADING')
          console.log(error)
          this.response = 'Server appears to be offline'
        })
    },
    methods: {
      imageUploaded (response) {
        console.log('response is', response)
        this.src = response.body.url
        this.imgUrl = response.body.url
        // alert(this.imgUrl)
        // this.imgUrl = 'https://upload.jianshu.io/users/upload_avatars/2204269/54bc6df9d4b6.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/240/h/240'
      },
      handleRemove (file, fileList) {
        this.imgUrl = ''
        console.log(file, fileList)
      },
      handlePreview (file) {
        console.log(file)
      },
      onEditorBlur (editor) {
        console.log('editor blur!', editor)
      },
      onEditorFocus (editor) {
        console.log('editor focus!', editor)
      },
      onEditorReady (editor) {
        console.log('editor ready!', editor)
      },
      onSubmit () {
        // alert(this.content)
        // this.$router.push('/org')
        // console.log('submit!')
        var iconId = '0'
        if (this.$route.query.iconId) {
          iconId = this.$route.query.iconId
          this.iconId = this.$route.query.iconId
        }
        var userid = localStorage.getItem('userid')
        console.log(userid)
        let formData = new FormData()
        formData.append('name', this.form.name)
        if (this.form.status === '正常') {
          formData.append('status', Number(0))
        } else {
          formData.append('status', Number(1))
        }
        formData.append('type', this.form.type)
        formData.append('targetId', this.form.targetId)
        formData.append('webUrl', this.form.webUrl)
        formData.append('iconId', Number(iconId))
        if (this.imgUrl !== '') {
          formData.append('coverUrl', this.imgUrl)
        }
        // formData.append('file', this.file)

        api.requestForm('post', 'icon/upload', formData)
          .then(response => {
            var data = response.data
            console.log(JSON.stringify(data))
            this.$message({
              message: '操作成功!',
              type: 'success'
            })
            //
            this.$router.go(-1)
          })
          .catch(error => {
            console.log(error)
            this.response = error
          })
      },
      handleImageAdded: function (file, Editor, cursorLocation) {
        // An example of using FormData
        // NOTE: Your key could be different such as:
        // formData.append('file', file)

        var formData = new FormData()
        formData.append('file', file)
        api.requestForm('post', 'file/upload', formData)
          .then((response) => {
            console.log('resultstr is', JSON.stringify(response.data.body))
            let url = response.data.body.url // Get url from response
            Editor.insertEmbed(cursorLocation, 'image', url)
          })
          .catch((err) => {
            console.log(err)
          })
      }
    }
  }
</script>

<style>
  .datetime-picker input {
    height: 4em !important;
  }
</style>
