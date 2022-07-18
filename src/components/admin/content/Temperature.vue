<template>
  <div>
    <el-row style="margin: 18px 0px 0px 18px ">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/admin/dashboard' }">管理中心</el-breadcrumb-item>
        <el-breadcrumb-item>数据展示</el-breadcrumb-item>
        <el-breadcrumb-item>温度信息</el-breadcrumb-item>
      </el-breadcrumb>
    </el-row>
<!--    <edit-form @onSubmit="loadBooks()" ref="edit"></edit-form>-->
    <el-card style="width: 95%;margin: 40px 2% 18px;">
      <div id="borrow-history">
        <p style="margin-right: 1100px">
          <span><i class="el-icon-s-fold"></i></span>
          <span style="font-size: 20px">历史温度查询</span>
        </p>
        <div>
          <span style="margin-left: 500px">
            <el-input v-model="room" placeholder="按房间查询" style="width: 150px"></el-input>
          </span>
          <span style="margin-left: 10px">
            <el-date-picker
              v-model="date"
              type="daterange"
              align="right"
              unlink-panels
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              value-format="yyyy-MM-dd"
              :picker-options="pickerOptions">
              </el-date-picker>
              <el-button @click="loadBorrowHistoryAccordingDateAndUsername()">查询</el-button>
          </span>
        </div>
        <el-table
          :data="borrowHistory"
          style="width: 100%;margin-top: 20px"
          max-height="500">
          <el-table-column
            prop="username"
            label="房间"
            fit>
          </el-table-column>
          <el-table-column
            prop="title"
            label="传感器编号"
            fit>
          </el-table-column>
          <el-table-column
            prop="startdate"
            label="平均温度"
            class="temperatureColumn">
          </el-table-column>
          <el-table-column
            prop="enddate"
            label="最高温度"
            class="temperatureColumn">
          </el-table-column>
          <el-table-column
            prop="returndate"
            label="最低温度"
            class="temperatureColumn">
          </el-table-column>
        </el-table>
      </div>
    </el-card>
  </div>
</template>

<script>
  export default {
    name: 'TemperatureManagement',
    components: { },
    data () {
      return {
        books: [],
        date: '',
        room: '',
        pickerOptions: {
          shortcuts: [{
            text: '最近一周',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
              picker.$emit('pick', [start, end])
            }
          }, {
            text: '最近一个月',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
              picker.$emit('pick', [start, end])
            }
          }, {
            text: '最近三个月',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
              picker.$emit('pick', [start, end])
            }
          }]
        }
      }
    },
    mounted () {
      this.loadBooks()
    },
    computed: {
      tableHeight () {
        return window.innerHeight - 250
      }
    },
    methods: {
      deleteBook (id) {
        this.$confirm('此操作将永久删除该书籍, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
            this.$axios
              .post('/admin/content/books/delete', {id: id}).then(resp => {
              if (resp && resp.data.code === 200) {
                this.loadBooks()
              }
            })
          }
        ).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
      },
      editBook (item) {
        this.$refs.edit.dialogFormVisible = true
        this.$refs.edit.form = {
          id: item.id,
          cover: item.cover,
          title: item.title,
          author: item.author,
          date: item.date,
          press: item.press,
          abs: item.abs,
          category: {
            id: item.category.id.toString(),
            name: item.category.name
          }
        }
        // this.$refs.edit.category = {
        //   id: item.category.id.toString()
        // }
      },
      loadBooks () {
        var _this = this
        this.$axios.get('/books').then(resp => {
          if (resp && resp.data.code === 200) {
            _this.books = resp.data.result
          }
        })
      },
      loadBorrowHistoryAccordingDate () { // 按照时间加载借阅历史
        var _this = this
        this.$axios.post('/admin/borrow_history_all_accordingDate', {
          date: this.date
        }).then(resp => {
          if (resp && resp.data.code === 200) {
            _this.borrowHistory = resp.data.result
          }
        })
      },
      loadBorrowHistoryAccordingUsername () { // 按照用户名加载借阅历史
        var _this = this
        this.$axios.post('/admin/borrow_history_all_accordingUsername', {
          username: this.username
        }).then(resp => {
          if (resp && resp.data.code === 200) {
            _this.borrowHistory = resp.data.result
          }
        })
      },
      loadBorrowHistoryAccordingDateAndUsername () { // 根据日期和用户名加载借阅历史
        if (this.room === '' && (this.date === null || this.date === '')) {
          this.loadBooks()
        } else if (this.room !== '' && (this.date === null || this.date === '')) {
          this.loadBorrowHistoryAccordingUsername()
        } else if (this.room === '' && this.date !== null && this.date !== '') {
          this.loadBorrowHistoryAccordingDate()
        } else {
          var _this = this
          this.$axios.post('/admin/borrow_history_all_accordingDateAndUsername', {
            username: this.username,
            startdate: this.date[0],
            returndate: this.date[1]
          }).then(resp => {
            if (resp && resp.data.code === 200) {
              _this.borrowHistory = resp.data.result
            }
          })
        }
      }
    }
  }
</script>

<style scoped>
  .temperatureColumn {
    width: 280px;
  }
</style>
