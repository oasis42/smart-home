<template>
  <div>
    <el-row style="margin: 18px 0px 0px 18px ">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/admin/dashboard' }">管理中心</el-breadcrumb-item>
        <el-breadcrumb-item>数据展示</el-breadcrumb-item>
        <el-breadcrumb-item>湿度信息</el-breadcrumb-item>
      </el-breadcrumb>
    </el-row>
    <el-card style="width: 95%;margin: 40px 2% 18px;">
      <div id="borrow-history">
        <p style="margin-right: 1100px">
          <span><i class="el-icon-s-fold"></i></span>
          <span style="font-size: 20px">历史湿度查询</span>
        </p>
        <div>
          <span style="margin-left: 500px">
<!--            <el-input v-model="username" placeholder="按房间查询" style="width: 150px"></el-input>-->
            <el-select v-model="room" placeholder="按房间查询"  >
              <el-option  label="文学" value="1" key="文学"></el-option>
              <el-option label="流行" value="2"></el-option>
              <el-option label="文化" value="3"></el-option>
              <el-option label="生活" value="4"></el-option>
              <el-option label="经管" value="5"></el-option>
              <el-option label="科技" value="6"></el-option>
            </el-select>
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
              <el-button @click="loadRecordAccordingDateAndRoom()">查询</el-button>
          </span>
        </div>
        <el-table
          :data="books"
          style="width: 100%;margin-top: 20px"
          max-height="300">
          <el-table-column
            prop="title"
            label="房间"
            fit>
          </el-table-column>
          <el-table-column
            prop="category.name"
            label="传感器编号"
            class="humidityColumn">
          </el-table-column>
          <el-table-column
            prop="author"
            label="平均湿度"
            class="humidityColumn">
          </el-table-column>
          <el-table-column
            prop="author"
            label="最高湿度"
            class="humidityColumn">
          </el-table-column>
          <el-table-column
            prop="author"
            label="最低湿度"
            class="humidityColumn">
          </el-table-column>
        </el-table>
      </div>
    </el-card>
    <panel-group @handleSetLineChartData="handleSetLineChartData" />
    <el-card style="background:#fff;margin:40px 2% 18px;width: 95%;">
      <line-chart :chart-data="lineChartData" />
    </el-card>
  </div>
</template>

<script>
import LineChart from './components/LineChart'

var lineChartData = {
  newVisitis: [120, 82, 91, 154, 162, 140, 145],
  date: [],
  category: []
}
    export default {
      name: 'HumidityManagement',
      components: {
        LineChart
      },
      data () {
        return {
          books: [],
          date: '',
          room: '',
          lineChartData: lineChartData,
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
        this.loadRecord()
      },
      computed: {
        tableHeight () {
          return window.innerHeight - 250
        }
      },
      methods: {
        loadRecord () {
          var _this = this
          this.$axios.get('/books').then(resp => {
            if (resp && resp.data.code === 200) {
              _this.books = resp.data.result
              // 先清空
              lineChartData.date.length = 0
              lineChartData.category.length = 0
              resp.data.result.forEach((item) => {
                // 后面改成Date和湿度
                lineChartData.date.push(item.id)
                lineChartData.category.push(item.category.id)
              })
            }
          })
        },
        loadRecordAccordingDate () { // 按照时间加载记录
          var _this = this
          this.$axios.post('/admin/borrow_history_all_accordingDate', {
            date: this.date
          }).then(resp => {
            if (resp && resp.data.code === 200) {
              _this.borrowHistory = resp.data.result
            }
          })
        },
        loadRecordAccordingRoom () { // 按照房间加载记录
          var _this = this
          this.$axios.post('/admin/borrow_history_all_accordingUsername', {
            username: this.username
          }).then(resp => {
            if (resp && resp.data.code === 200) {
              _this.borrowHistory = resp.data.result
            }
          })
        },
        loadRecordAccordingDateAndRoom () { // 根据房间和日期加载历史记录
          debugger
          if (this.room === '' && (this.date === null || this.date === '')) {
            this.loadRecord()
          } else if (this.room !== '' && (this.date === null || this.date === '')) {
            this.loadRecordAccordingRoom()
          } else if (this.room === '' && this.date !== null && this.date !== '') {
            this.loadRecordAccordingDate()
          } else {
            var _this = this
            this.$axios.post('/admin/borrow_history_all_accordingDateAndUsername', {
              username: this.username,
              startdate: this.date[0],
              returndate: this.date[1]
            }).then(resp => {
              if (resp && resp.data.code === 200) {
                _this.borrowHistory = resp.data.result
              } else {
                _this.$router.replace('/admin/dashboard')
              }
            })
          }
        },
        handleSetLineChartData (type) {
          this.lineChartData = lineChartData[type]
        }
      }
    }
</script>

<style scoped>
  .humidityColumn {
    width: 280px;
  }
</style>
