<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 搜索与添加区域 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入书籍ID或订单编号" v-model="queryInfo.query" clearable @clear="clearMsg">
            <el-button slot="append" icon="el-icon-search" @click="getOrderList"></el-button>
          </el-input>
        </el-col>
      </el-row>
       <!-- 订单列表区域 -->
      <el-table
        :data="orderlist"
        style="width: 100%" border stripe>
        <el-table-column type="expand" fixed="left" label="#">
          <template slot-scope="props">
            <el-form   class="demo-table-expand">
              <el-form-item label="购买人ID">
                <span>{{ props.row.openid }}</span>
              </el-form-item>
              <el-form-item label="书籍ID">
                <span>{{ props.row.bookid }}</span>
              </el-form-item>
              <el-form-item label="购买数量">
                <span>{{ props.row.buyBookNum }}</span>
              </el-form-item>
              <el-form-item label="支付价格">
                <span>{{ props.row.spendPrice }}</span>
              </el-form-item>
              <el-form-item label="订单编号">
                <span>{{ props.row.orderNumber }}</span>
              </el-form-item>
              <el-form-item label="收件人">
                <span>{{ props.row.postname }}</span>
              </el-form-item>
              <el-form-item label="收货地址">
                <span>{{ props.row.adress }}</span>
              </el-form-item>
              <el-form-item label="收件人电话">
                <span>{{ props.row.telephone }}</span>
              </el-form-item>
            </el-form>
          </template>
        </el-table-column>
        <el-table-column type="index" label="序号"></el-table-column>
        <el-table-column label="购买人ID" prop="openid"></el-table-column>
        <el-table-column label="书籍ID" prop="bookid"></el-table-column>
        <el-table-column label="购买数量" prop="buyBookNum" width="80px"></el-table-column>
        <el-table-column label="支付价格" prop="spendPrice" width="80px"></el-table-column>
        <el-table-column label="订单编号" prop="orderNumber" width="150px"></el-table-column>
        <el-table-column label="购买时间" prop="buyTime" width="150px"></el-table-column>
        <el-table-column label="发货状态" width="100px">
          <template v-slot="scope">
            <el-tag type="success" v-if="scope.row.status === 0">已发货</el-tag>
            <el-tag type="warning" v-if="scope.row.status === 1">未发货</el-tag>
            <el-tag type="danger" v-if="scope.row.status === 2">取消发货</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="100px">
          <template v-slot="scope">
            <el-tooltip effect="dark" content="删除" placement="top" :enterable="false">
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteorder(scope.row.orderNumber)"></el-button>
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
        pagenum: 1,
        pagesize: 5
      },
      orderlist: [],
      total: 0
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('/orderlist', { params: this.queryInfo })
      this.orderlist = res
      this.total = res[0].total
    },
    // 监听pagesize改变的事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    // 监听页码值改变的事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    // 清空搜索栏
    clearMsg () {
      this.queryInfo.pagenum = 1
      this.queryInfo.pagesize = 5
      this.getOrderList()
    },
    // 删除订单
    async deleteorder (orderid) {
      const confirmResult = await this.$confirm('此操作将永久删除该订单, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.post('/deleteorder', { orderid })
      if (res.status === 502) {
        this.$message.success('删除成功')
        this.getOrderList()
      } else {
        this.$message.error('网络错误')
      }
    }
  }
}
</script>

<style lang="less" scoped>
.demo-table-expand {
    font-size: 0;
  }
  .demo-table-expand label {
    width: 90px;
    color: #99a9bf;
  }
  .demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
  }
</style>
