<template>
  <section class="content">

    <div class="row center-block" style="background: #ffffff">
      <div style="float:left;width: 40%;margin-left: 10px">
        <!--   <div style="margin-top: 15px;">
             <el-input placeholder="请输入内容" v-model="input5" class="input-with-select">

               <el-button slot="append" icon="el-icon-search">搜索</el-button>
             </el-input>
           </div>-->
        <table class="table table-bordered table-responsive table-striped" style="margin-top: 15px">
          <thead>
          <tr>
            <th style='text-align: center'>序号</th>
            <th style='text-align: center'>书名</th>
            <th style='text-align: center'>操作</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="(item,index) in arrayData" v-bind:key="item.name">
            <td style='text-align: center'>{{index + 1}}</td>
            <td style='text-align: center'>{{item.name}}</td>
            <td style='text-align: center;height: 50px'>
              <el-button type="text" @click="up(item.id,item.order,index)" >上移</el-button>
              <el-button type="text" @click="down(item.id,item.order,index)" >下移</el-button>
              <el-button type="text" @click="editUser(item.id)" style="visibility: hidden">编辑</el-button>
              <el-button type="text" @click="removeBookFromAlbum(item.id)">移除</el-button>
            </td>
          </tr>
          </tbody>
        </table>
        <div>
          <div align="center">
            <!--<span class="demonstration">调整每页显示条数</span>-->
            <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page.sync="currentPage"
              :page-sizes="[10, 20, 30, 40]"
              :page-size="20"
              layout="prev, pager, next"
              :total="totalCount">
            </el-pagination>
          </div>
        </div>
      </div>
      <div style="float:left;height: 700px;" id="myoutercontainer">
        <div id="myinnercontainer">
          <!--<div style="margin-left: 10px;margin-right: 10px" @onclick="moveLeft"><img src="/static/img/left.png"
                                                                                     style="width: 30px;height:30px">
          </div>
          <br>
          <div style="margin-left: 10px;margin-right: 10px"><img src="/static/img/right.png"
                                                                 style="width: 30px;height:30px"></div>-->

        </div>
      </div>
      <div style="float:left;width: 40%;margin-left:50px">
        <div style="margin-top: 15px;">
          <el-input placeholder="请输入内容" v-model="input5" class="input-with-select">
            <el-button slot="append" icon="el-icon-search" @click="search">搜索</el-button>
          </el-input>

        </div>
        <el-checkbox-group v-for="item in searchData" v-model="checkList">
          <el-checkbox :label="item.name" :value="item.id"></el-checkbox>
        </el-checkbox-group>
        <el-button type="primary" @click="moveLeft">添加选中书本到专辑</el-button>
      </div>

      <!-- /.box-body -->
    </div>

  </section>
</template>

<script>
  // import $ from 'jquery'
  import api from '../../api'
  import { formatDateBtk, formatStatus } from '../../filters/index.js'

  export default {
    filters: {
      BTKformatDate (time) {
        var date = new Date(time)
        return formatDateBtk(date, 'yyyy-MM-dd hh:mm')
      },
      FormatStatus (status) {
        return formatStatus(status)
      }
    },
    data () {
      return {
        currentPage: 1,
        // 为第一页或者最后一页时，首页，尾页不能点击
        fDisabled: false,
        lDisabled: false,
        // 总项目数
        totalCount: 40,
        // 分页数
        pageCount: 20,
        // 当前页面
        pageCurrent: 1,
        // 分页大小
        pagesize: 20,
        // 显示分页按钮数
        showPages: 11,
        // 开始显示的分页按钮
        showPagesStart: 1,
        // 结束显示的分页按钮
        showPageEnd: 100,
        // 分页数据
        arrayData: [],
        searchData: [],
        // 删除确认框
        dialogVisible: false,
        dialogTableVisible: false,
        checkList: [],
        input5: ''
      }
    },
    methods: {
      up (id, order, index) {
        // 上移一位
        if (index > 0) {
          var newId = this.arrayData[index - 1].id
          var newOrder = this.arrayData[index - 1].order
          var params = {
            'bookId': Number(-11),
            'actionCode': 8,
            'bookIds': '',
            'albumIds': '',
            'subCategoryId': Number(-11),
            'originId': id,
            'originOrder': order,
            'newId': newId,
            'newOrder': newOrder
          }
          api.request('post', '/relation/book/taglist/edit', params)
            .then(response => {
              this.init()
            })
            .catch(error => {
              console.log(error)
              this.response = error
            })
        }
      },
      down (id, order, index) {
        // 下移一位
        if (index < 20) {
          var newId = this.arrayData[index + 1].id
          var newOrder = this.arrayData[index + 1].order
          var params = {
            'bookId': Number(-11),
            'actionCode': 8,
            'bookIds': '',
            'albumIds': '',
            'subCategoryId': Number(-11),
            'originId': id,
            'originOrder': order,
            'newId': newId,
            'newOrder': newOrder
          }
          api.request('post', '/relation/book/taglist/edit', params)
            .then(response => {
              this.init()
            })
            .catch(error => {
              console.log(error)
              this.response = error
            })
        }
      },
      search () {
        // var userid = localStorage.getItem('userid')
        var albumId = '0'
        if (this.$route.query.albumId) {
          albumId = this.$route.query.albumId
          this.albumId = this.$route.query.albumId
        }
        var keyvalue = this.input5
        if (keyvalue.length === 0) {
          keyvalue = ' '
        }
        api.request('get', 'book/list?type=1&userid=1&pageSize=20&pageIndex=0&keyword=' + keyvalue + '&albumId=' + albumId)
          .then(response => {
            this.searchData = response.data.body.bookList
            this.totalCount = response.data.body.status.totalCount
          })
          .catch(error => {
            console.log(error)
            this.response = 'Server appears to be offline'
          })
      },
      handleSizeChange (val) {
        console.log(`每页 ${val} 条`)
      },
      moveLeft () {
        // alert('ok')
        var booksId = ''
        for (var i = 0; i < this.checkList.length; i++) {
          for (var k = 0; k < this.searchData.length; k++) {
            if (this.checkList[i] === this.searchData[k].name) {
              if (booksId === '') {
                booksId = this.searchData[k].id + ''
              } else {
                booksId = booksId + ',' + this.searchData[k].id
              }
            }
          }
        }
        // alert(booksId)
        // 提交
        var albumId = 0
        if (this.$route.query.albumId) {
          albumId = this.$route.query.albumId
          this.albumId = this.$route.query.albumId
        }
        console.log(albumId)
        var params = {
          'albumId': Number(albumId),
          'actionCode': Number(0),
          'action': Number(0),
          'bookIds': booksId
        }
        api.request('post', '/relation/album/booklist/edit', params)
          .then(response => {
            // var data = response.data
            // alert(JSON.stringify(data))
            this.search()
            this.init()
          })
          .catch(error => {
            console.log(error)
            this.response = error
          })
      },
      removeBookFromAlbum (bookId) {
        this.$confirm('此操作将永久删除 ' + ', 是否继续?', '提示', {type: 'warning'})
          .then(() => {
            // 向请求服务端删除
            var albumId = 0
            if (this.$route.query.albumId) {
              albumId = this.$route.query.albumId
              this.albumId = this.$route.query.albumId
            }
            var userid = localStorage.getItem('userid')
            var params = {
              'userId': userid,
              'albumId': Number(albumId),
              'actionCode': 1,
              'action': 1,
              'bookIds': bookId + ''
            }
            api.request('post', '/relation/album/booklist/edit', params)
              .then(response => {
                // var data = response.data
                // alert(JSON.stringify(data))
                this.search()
                this.init()
              })
              .catch(error => {
                console.log(error)
                this.response = error
              })
          })
          .catch(() => {
            this.$message.info('已取消操作!')
          })
      },
      handleCurrentChange (val) {
        console.log(`当前页: ${val}`)
        var userid = localStorage.getItem('userid')
        api.request('post', 'book/list?userid=' + userid + '&pageIndex=' + (Number(val) - 1) + '&pageSize=20')
          .then(response => {
            console.log(response.data)
            this.arrayData = response.data.body.bookList
            this.totalCount = response.data.body.status.totalCount
          })
          .catch(error => {
            // this.$store.commit('TOGGLE_LOADING')
            console.log(error)
            this.response = 'Server appears to be offline'
          })
      },
      editUser (userId) {
        // this.$router.push('/org/edit?orgid=' + agentId)
        this.$router.push({path: '/user/edit?userid=' + userId})
      },
      init () {
        // alert('init')
        var albumId = '0'
        if (this.$route.query.albumId) {
          albumId = this.$route.query.albumId
          this.albumId = this.$route.query.albumId
        }
        // var userid = localStorage.getItem('userid')
        api.request('get', 'book/list?userid=1&pageSize=20&pageIndex=0&albumId=' + albumId)
          .then(response => {
            this.arrayData = response.data.body.bookList
            this.totalCount = response.data.body.status.totalCount
          })
          .catch(error => {
            console.log(error)
            this.response = 'Server appears to be offline'
          })
      }
    },
    created () {
      this.init()
    },
    mounted () {
      // this.showPage(this.pageCurrent, null, true)
    },
    computed: {}
  }

</script>

<style>
  /* Using the bootstrap style, but overriding the font to not draw in
     the Glyphicons Halflings font as an additional requirement for sorting icons.

     An alternative to the solution active below is to use the jquery style
     which uses images, but the color on the images does not match adminlte.

  @import url('/static/js/plugins/datatables/jquery.dataTables.min.css');
  */
  @import url('/static/js/plugins/datatables/dataTables.bootstrap.css');

  table.dataTable thead .sorting:after,
  table.dataTable thead .sorting_asc:after,
  table.dataTable thead .sorting_desc:after {
    font-family: 'FontAwesome';
  }

  table.dataTable thead .sorting:after {
    content: "\f0dc";
  }

  table.dataTable thead .sorting_asc:after {
    content: "\f0dd";
  }

  table.dataTable thead .sorting_desc:after {
    content: "\f0de";
  }

  .el-select .el-input {
    width: 130px;
  }

  .input-with-select .el-input-group__prepend {
    background-color: #fff;
  }

  #myoutercontainer {
    position: relative
  }

  #myinnercontainer {
    position: absolute;
    top: 50%;
    height: 10em;
    margin-top: -5em
  }
</style>
