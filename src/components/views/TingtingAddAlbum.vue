<template>
  <div>
    <h5 class="text-center">添加专辑</h5>
    <section class="content">
      <div class="row">
        <div class="col-md-12">
          <el-form ref="form" :model="form" label-width="80px">
            <el-form-item label="标题">
              <el-input v-model="form.name"></el-input>
            </el-form-item>
            <!--<el-form-item label="年级">
              <el-input v-model="form.grade"></el-input>
            </el-form-item>-->

            <el-form-item label="定价">
              <el-input-number v-model="form.value" @change="handleChange" :min="0" :max="10000"
                               label="改变价格"></el-input-number>
            </el-form-item>

            <el-form-item label="图标">
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
            <el-form-item label="年级">
              <!--<el-input v-model="form.grade"></el-input>-->
              <el-tree
                :data="data2"
                show-checkbox
                default-expand-all
                node-key="id"
                ref="tree"
                highlight-current
                :props="defaultProps">
              </el-tree>
            </el-form-item>
            <!-- bidirectional data binding（双向数据绑定） -->


            <el-form-item label="简介">
              <vue-editor id="editor"
                          useCustomImageHandler
                          @imageAdded="handleImageAdded" v-model="htmlForEditor">
              </vue-editor>
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
        htmlForEditor: '',
        uploadUrl: 'http://192.168.200.208:81/nanjingyouzi/TingtingBackend/1.0.0/file/upload',
        form: {
          value: '',
          name: '',
          region: '',
          date1: '',
          date2: '',
          delivery: false,
          type: [],
          resource: '',
          desc: '',
          grade: '13',
          status: '',
          content: '',
          fileList2: [{
            name: 'food.jpeg',
            url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'
          }]
        },
        data2: [{
          id: 10,
          label: '学龄前'
        }, {
          id: 20,
          label: '幼儿园'
        }, {
          id: 30,
          label: '小学',
          children: [{
            id: 31,
            label: '小学一年级'
          }, {
            id: 32,
            label: '小学二年级'
          }, {
            id: 33,
            label: '小学三年级'
          }, {
            id: 34,
            label: '小学四年级'
          }, {
            id: 35,
            label: '小学五年级'
          }, {
            id: 36,
            label: '小学六年级'
          }]
        }, {
          id: 40,
          label: '初中',
          children: [{
            id: 41,
            label: '初中一年级'
          }, {
            id: 42,
            label: '初中二年级'
          }, {
            id: 43,
            label: '初中三年级'
          }]
        }, {
          id: 50,
          label: '高中',
          children: [{
            id: 51,
            label: '高中一年级'
          }, {
            id: 52,
            label: '高中二年级'
          }, {
            id: 53,
            label: '高中三年级'
          }]
        }, {
          id: 60,
          label: '大学',
          children: [{
            id: 61,
            label: '大学一年级'
          }, {
            id: 62,
            label: '大学二年级'
          }, {
            id: 63,
            label: '大学三年级'
          }, {
            id: 64,
            label: '大学四年级'
          }]
        }],
        defaultProps: {
          children: 'children',
          label: 'label'
        },
        name: '01-example',
        content: '',
        imgUrl: '',
        content1: ''
      }
    },
    computed: {
      editor () {
        return this.$refs.myTextEditor.quill
      },
      contentCode () {
        // return hljs.highlightAuto(this.content).value
      }
    },
    created () {
      this.uploadUrl = configParams.uploadURI
    },
    methods: {
      imageUploaded (response) {
        console.log('response is', response)
        this.src = response.body.url
        this.imgUrl = response.body.url
        // alert(this.imgUrl)
        // this.imgUrl = 'https://upload.jianshu.io/users/upload_avatars/2204269/54bc6df9d4b6.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/240/h/240'
      },
      successUpload (response, file, fileList) {
        console.log('response is ' + JSON.stringify(response))
        this.imgUrl = response.body.url
        console.log(this.imgUrl)
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
        var userid = localStorage.getItem('userid')
        console.log(userid)
        let formData = new FormData()
        formData.append('name', this.form.name)
        if (this.form.status === '正常') {
          formData.append('status', Number(0))
        } else {
          formData.append('status', Number(1))
        }
        formData.append('subTitle', this.form.name)
        formData.append('title', this.form.name)
        formData.append('albumId', Number(-1))
        formData.append('grade', Number(this.form.grade))
        formData.append('summary', this.htmlForEditor)
        formData.append('gradeRange', this.$refs.tree.getCheckedKeys())
        formData.append('priceValue', this.form.value)
        if (this.imgUrl !== '') {
          formData.append('iconUrl', this.imgUrl)
        }
        // formData.append('file', this.file)

        api.requestForm('post', 'album/upload', formData)
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

  .quill-code {
    border: none;
    height: auto;
  }

  > code {
    width: 100%;
    margin: 0;
    padding: 1rem;
    border: 1px solid #ccc;
    border-top: none;
    border-radius: 0;
    height: 10rem;
    overflow-y: auto;
    resize: vertical;
  }

</style>
