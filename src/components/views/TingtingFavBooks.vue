<template>
  <section class="content">

    <div class="row center-block" style="background: #ffffff">
      <div id="example1_length" class="dataTables_length">
        <!--<router-link class="pageLink" to="/album/add">
          <a>
            <span class="page" style="float:right;margin:5px"><el-button type="success" plain>添加专辑</el-button></span>

          </a>
        </router-link>-->

      </div>
      <table class="table table-bordered table-responsive table-striped">
        <thead>
        <tr>
          <th style='text-align: center'>序号</th>
          <th style='text-align: center'>书名</th>
          <th style='text-align: center'>图标</th>
          <!--<th style='text-align: center'>书本数</th>
          <th>副标题</th>-->
          <!--<th>是否显示icon</th>-->
          <!--<th>大图
          <th style='text-align: center'>播放数</th>
          <th style='text-align: center'>价格</th>
          <th style='text-align: center'>年级</th></th>-->
          <th style='text-align: center'>收藏时间</th>
          <!--<th>更新提示</th>-->
          <!-- <th>播放地址</th>
           <th>顺序</th>
           <th>状态</th>
           <th>时间</th>
          <th style='text-align: center'>操作</th>-->
        </tr>
        </thead>
        <tbody>
        <tr v-for="(item,index) in arrayData" v-bind:key="item.name">
          <td style='text-align: center'>{{index + 1}}</td>
          <td style='text-align: center'>{{item.name}}</td>
          <td style='text-align: center'><img v-bind:src=item.icon style="width: 40px;height:40px"></td>
          <!--<td style='text-align: center'>{{item.booksNumber}}</td>
          <td class="sorting_1" style="vertical-align: middle">{{item.subTitle}}</td>-->
          <!--<td class="sorting_1" style="vertical-align: middle">{{item.showIcon}}</td>-->
          <!--<td class="sorting_1" style="vertical-align: middle">{{item.bigCover}}</td>
          <td style='text-align: center'>{{item.playCount}}</td>
          <td style='text-align: center'>{{item.value}}</td>
          <td style='text-align: center'>{{item.grade}}</td>-->
          <td class="sorting_1" style="vertical-align: middle">{{item.time * 1000 | BTKformatDate}}</td>
          <!--<td class="sorting_1" style="vertical-align: middle">{{item.duration}}</td>
          &lt;!&ndash;<td class="sorting_1" style="vertical-align: middle">{{item.updateTips}}</td>&ndash;&gt;
          <td class="sorting_1" style="vertical-align: middle">{{item.url}}</td>
          <td class="sorting_1" style="vertical-align: middle">{{item.order}}</td>
          <td class="sorting_1" style="vertical-align: middle">{{item.status | FormatStatus}}</td>
          <td class="sorting_1" style="vertical-align: middle">{{item.time*1000 | BTKformatDate}}</td>
          <td style='text-align: center'>

            <el-button type="success" round @click="editRelation(item.id)" plain>书本管理</el-button>
            <el-button type="primary" round @click="editTags(item.id)" plain>标签管理</el-button>
            <el-button type="success" round @click="sendPush(item.id,item.name)" plain>推送(已推{{item.times}}次)</el-button>
            <el-button type="info" round @click="editAlbum(item.id)" plain>编辑</el-button>
            <el-button type="warning" round @click="removeAlbum(item.id)" plain>删除</el-button>

          </td>-->
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
        // 删除确认框
        dialogVisible: false
      }
    },
    methods: {
      handleSizeChange (val) {
        console.log(`每页 ${val} 条`)
      },
      removeAlbum (albumId) {
        this.$confirm('此操作将永久删除 ' + ', 是否继续?', '提示', {type: 'warning'})
          .then(() => {
            // 向请求服务端删除
            var userid = localStorage.getItem('userid')
            api.request('get', 'album/delete?albumId=' + albumId + '&operator_id=' + userid)
              .then(response => {
                console.log(response.data)
                this.$message.info('删除成功!')
                // reload
                api.request('get', 'book/list?operator_id=' + userid + '&pageSize=20&pageIndex=0')
                  .then(response => {
                    this.totalCount = response.data.body.status.totalCount
                    this.arrayData = response.data.body.albumList
                  })
                  .catch(error => {
                    // this.$store.commit('TOGGLE_LOADING')
                    console.log(error)
                    this.response = error
                  })
              })
              .catch(error => {
                // this.$store.commit('TOGGLE_LOADING')
                console.log(error)
                this.response = error
              })
          })
          .catch(() => {
            this.$message.info('已取消操作!')
          })
      },
      sendPush (albumId, name) {
        this.$confirm('此操作将向客户端发送一条专辑《' + name + '》推送通知 ' + ', 是否继续?', '提示', {type: 'warning'})
          .then(() => {
            // 向请求服务端删除
            var userid = localStorage.getItem('userid')
            api.request('get', 'push/jpush?title=' + name + '&type=3&id=' + albumId + '&operator_id=' + userid)
              .then(response => {
                console.log(response.data)
                this.$message.info('发送成功!')
                // reload
                api.request('get', 'book/list?userid=1&pageSize=20&pageIndex=0')
                  .then(response => {
                    this.totalCount = response.data.body.status.totalCount
                    this.arrayData = response.data.body.albumList
                  })
                  .catch(error => {
                    // this.$store.commit('TOGGLE_LOADING')
                    console.log(error)
                    this.response = error
                  })
              })
              .catch(error => {
                // this.$store.commit('TOGGLE_LOADING')
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
        api.request('get', 'book/list?userid=' + userid + '&pageIndex=' + (Number(val) - 1) + '&pageSize=20')
          .then(response => {
            console.log(response.data)
            this.arrayData = response.data.body.albumList
          })
          .catch(error => {
            // this.$store.commit('TOGGLE_LOADING')
            console.log(error)
            this.response = 'Server appears to be offline'
          })
      }
    },
    created () {
      var memberId = '0'
      if (this.$route.query.memberId) {
        memberId = this.$route.query.memberId
        this.memberId = this.$route.query.memberId
      }
      // var userid = localStorage.getItem('userid')
      api.request('get', 'book/list?userid=1&pageSize=20&pageIndex=0&memberId=' + memberId)
        .then(response => {
          console.log(response.data)
          this.totalCount = response.data.body.status.totalCount
          this.arrayData = response.data.body.bookList
          for (var i = 0; i < this.arrayData.length; i++) {
            this.arrayData.time = formatDateBtk(this.arrayData.time)
            // this.arrayData.last_time = formatDateBtk(this.arrayData.last_time)
            console.log()
          }
        })
        .catch(error => {
          // this.$store.commit('TOGGLE_LOADING')
          console.log(error)
          this.response = 'Server appears to be offline'
        })
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
</style>
