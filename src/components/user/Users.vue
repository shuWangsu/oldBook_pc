<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 搜索与添加区域 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="clearMsg">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 书籍列表区域 -->
      <el-table :data="userslist" border stripe>
        <el-table-column type="index" label="序号"></el-table-column>
        <el-table-column label="用户id" prop="openid"></el-table-column>
        <el-table-column label="用户昵称" prop="wxusername"></el-table-column>
        <el-table-column label="是否禁用" width="100px">
          <template v-slot="scope">
            <el-switch
              v-model="scope.row.status"
              active-color="#13ce66"
              inactive-color="#ff4949" @change="userStatusChange(scope.row.openid,scope.row.status)">
            </el-switch>
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
        pagenum: 1,
        pagesize: 5
      },
      // 用户列表
      userslist: [],
      total: 0
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('/users', { params: this.queryInfo })
      this.userslist = res
      for (var i = 0; i < this.userslist.length; i++) {
        if (this.userslist[i].status === 1) {
          this.userslist[i].status = true
        } else {
          this.userslist[i].status = false
        }
      }
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
    // switch状态改变
    async userStatusChange (id, status) {
      if (status) {
        status = 1
      } else {
        status = 0
      }
      const { data: res } = await this.$http.post('/changestatus', { openid: id, status })
      if (res.status === 502) {
        if (status === 1) {
          this.$message.success('禁用用户成功')
        } else {
          this.$message.success('已取消禁用')
        }
      } else {
        this.$message.error('网络错误')
      }
    }
  }
}
</script>

<style lang="less" scoped>
.el-table {
  width:55%;
}
</style>
