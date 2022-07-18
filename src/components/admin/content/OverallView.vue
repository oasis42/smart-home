<template>
  <div>
    <el-row style="margin: 18px 0px 0px 18px ">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/admin/dashboard' }">管理中心</el-breadcrumb-item>
        <el-breadcrumb-item>数据展示</el-breadcrumb-item>
        <el-breadcrumb-item>总体信息</el-breadcrumb-item>
      </el-breadcrumb>
    </el-row>
    <el-card style="margin: 18px 2%;width: 95%">
      <el-table
        :data="books"
        stripe
        style="width: 100%"
        :max-height="tableHeight">
        <el-table-column
          prop="title"
          label="房间"
          width="300">
        </el-table-column>
        <el-table-column
          prop="category.name"
          label="当前温度"
          width="300">
        </el-table-column>
        <el-table-column
          prop="author"
          label="当前湿度"
          width="300">
        </el-table-column>
        <el-table-column
          prop="author"
          label="房间是否起火"
          fit>
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          fit>
          <template slot-scope="scope">
            <el-button
              @click.native.prevent="editBook(scope.row)"
              type="text">
              断电
            </el-button>
            <el-button
              @click.native.prevent="deleteBook(scope.row.id)"
              type="text">
              供电
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'OverallManagement',
  components: { },
  data () {
    return {
      books: []
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
    loadBooks () {
      var _this = this
      this.$axios.get('/books').then(resp => {
        if (resp && resp.data.code === 200) {
          _this.books = resp.data.result
        }
      })
    }
  }
}
</script>

<style scoped>
</style>
