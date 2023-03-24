<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>电影管理</el-breadcrumb-item>
      <el-breadcrumb-item>电影列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getMoviesList">
            <el-button slot="append" icon="el-icon-search" @click="getMoviesList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加电影</el-button>
        </el-col>
      </el-row>

      <!-- table表格区域 -->
      <el-table :data="movieslist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="电影名称" prop="name" width="110px"></el-table-column>
        <el-table-column label="导演" prop="director" width="110px"></el-table-column>
        <el-table-column label="演员" prop="actor" width="250px"></el-table-column>
        <el-table-column label="类型" prop="type" width="140px"></el-table-column>
        <el-table-column label="地区" prop="area" width="50px"></el-table-column>
        <el-table-column label="语言" prop="language" width="55px"></el-table-column>
        <el-table-column label="上映时间" prop="startTime" width="90px">
          <template slot-scope="scope">
            {{scope.row.startTime | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="时长/min" prop="timeLength" width="75px"></el-table-column>
        <el-table-column label="评分" prop="score" width="55px"></el-table-column>
        <el-table-column label="操作" width="140px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.id)"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[2, 10, 15, 20]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total" background>
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 商品列表
      movieslist: [],
      // 总数据条数
      total: 0
    }
  },
  created() {
    this.getMoviesList()
  },
  methods: {
    // 根据分页获取对应的商品列表
    async getMoviesList() {
      const { data: res } = await this.$http.post('movieInfo/list', this.queryInfo)

      if (res.status !== 200) {
        return this.$message.error('获取电影列表失败！')
      }

      this.$message.success('获取电影列表成功！')
      // console.log(res.data)
      this.movieslist = res.data.rows
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getMoviesList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getMoviesList()
    },
    async removeById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该电影, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消删除！')
      }

      const { data: res } = await this.$http.delete(`movieInfo/${id}`)

      if (res.status !== 200) {
        return this.$message.error('删除失败！')
      }

      this.$message.success('删除成功！')
      await this.getMoviesList()
    },
    goAddpage() {
      this.$router.push('/movies/add')
    }
  }
}
</script>

<style lang="less" scoped>
</style>
