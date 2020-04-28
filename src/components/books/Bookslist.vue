<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>书籍管理</el-breadcrumb-item>
      <el-breadcrumb-item>书籍列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card  class="box-card">
      <!-- 搜索与添加区域 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="clearMsg">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 书籍列表区域 -->
      <el-table :data="bookslist" border stripe>
        <el-table-column type="index" label="序号" align="center"></el-table-column>
        <el-table-column label="书籍名称" prop="bookname" align="center"></el-table-column>
        <el-table-column label="发布id" prop="openid" align="center"></el-table-column>
        <el-table-column label="书籍作者" prop="bookauthor" align="center"></el-table-column>
        <el-table-column label="剩余数量" prop="booknum" align="center"></el-table-column>
        <el-table-column label="售卖价格" prop="bookprice" align="center"></el-table-column>
        <el-table-column label="联系方式" prop="usertel" align="center"></el-table-column>
        <el-table-column label="发布时间" prop="uptime" align="center"></el-table-column>
        <el-table-column label="操作" align="center">
          <template v-slot="scope">
            <el-tooltip effect="dark" content="删除" placement="top" :enterable="false">
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteBook(scope.row.bookid)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        // 当前的页数
        pagenum: 1,
        // 当前每页显示多少条数据
        pagesize: 5
      },
      // 书籍列表
      bookslist: [],
      total: 20
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('/bookslist', { params: this.queryInfo })
      this.bookslist = res
      this.total = res[0].total
    },
    // 监听pagesize改变的事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听页码值改变的事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 清空搜索栏
    clearMsg () {
      this.queryInfo.pagenum = 1
      this.queryInfo.pagesize = 5
      this.getUserList()
    },
    // 删除书籍
    async deleteBook (id) {
      var bookid = id
      const confirmResult = await this.$confirm('此操作将永久删除该书籍, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.get('/deletebook', { params: { bookid } })
      if (res.status === 502) {
        this.$message.success('删除成功')
        this.getUserList()
      } else {
        this.$message.error('网络错误')
      }
    }
  }
}
</script>

<style lang="less" scoped>
</style>
