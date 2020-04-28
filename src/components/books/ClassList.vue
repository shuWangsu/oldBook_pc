<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>书籍管理</el-breadcrumb-item>
      <el-breadcrumb-item>类别列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入书籍类别" v-model="queryInfo.query" clearable @clear="clearMsg">
            <el-button slot="append" icon="el-icon-search" @click="getClassList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="8">
          <el-button type="primary" @click="addBookClass">添加类别</el-button>
        </el-col>
      </el-row>
      <!-- 书籍列表区域 -->
      <el-table :data="classlist" border stripe>
        <el-table-column type="index" label="序号" align="center"></el-table-column>
        <el-table-column label="类别名称" prop="className" align="center" ></el-table-column>
        <el-table-column label="操作" align="center">
          <template v-slot="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="editBookClass(scope.row)"></el-button>
            <el-tooltip effect="dark" content="删除" placement="top" :enterable="false">
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteClass(scope.row.classId)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[1, 5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
    <!-- 添加书籍类别 -->
    <el-dialog
      title="添加书籍类别"
      :visible.sync="addDialogVisible"
      width="50%" @close="closedDialog">
      <!-- 内容主体区域 -->
      <el-form :model="addForm"
      :rules="addFormRules" ref="addFormRef" label-width="80px">
        <el-form-item label="类别名称" prop="classname">
          <el-input v-model="addForm.classname"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="goAddBook">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑书籍类别 -->
    <el-dialog
      title="编辑书籍类别"
      :visible.sync="editDialogVisible"
      width="50%" @close="closedDialog">
      <!-- 内容主体区域 -->
      <el-form :model="addForm"
      :rules="addFormRules" ref="addFormRef" label-width="80px">
        <el-form-item label="类别名称" prop="classname">
          <el-input v-model="addForm.classname"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="goEditClass">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 获取书籍类别列表的参数对象
      queryInfo: {
        query: '',
        // 当前的页数
        pagenum: 1,
        // 当前每页显示多少条数据
        pagesize: 5
      },
      // 书籍列表
      classlist: [],
      total: 15,
      // 控制添加书籍类别对话框的显示与隐藏
      addDialogVisible: false,
      // 控制编辑书籍类别的对话框显示与否
      editDialogVisible: false,
      // 添加书籍的书籍
      addForm: {
        classname: ''
      },
      // 添加表单验证规则
      addFormRules: {
        classname: [{ required: true, message: '请输入类别名称', trigger: 'blur' },
          { min: 2, max: 5, message: '类别名称长度应在2到5个字符之间', trigger: 'blur' }]
      },
      // 存放classID
      classId: 0
    }
  },
  created () {
    this.getClassList()
  },
  methods: {
    async getClassList () {
      const { data: res } = await this.$http.get('/classlist', { params: this.queryInfo })
      this.classlist = res
      this.total = res[0].total
    },
    // 删除书籍
    async deleteClass (id) {
      var classId = id
      const confirmResult = await this.$confirm('此操作将永久删除该类别, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.get('/deletebookclass', { params: { classId } })
      if (res.status === 502) {
        this.$message.success('删除成功')
        this.getClassList()
      } else {
        this.$message.error('网络错误')
      }
    },
    // 监听pagesize改变的事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getClassList()
    },
    // 监听页码值改变的事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getClassList()
    },
    // 清空搜索栏
    clearMsg () {
      this.queryInfo.pagenum = 1
      this.queryInfo.pagesize = 5
      this.getClassList()
    },
    // 显示编辑书籍种类对话框
    editBookClass (data) {
      this.editDialogVisible = true
      this.addForm.classname = data.className
      this.classId = data.classId
    },
    // 提交编辑信息
    goEditClass () {
      var classId = this.classId
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        var classname = this.addForm.classname
        const { data: res } = await this.$http.post('/editbookclass', { classname: classname, classId })
        if (res.status !== 502) {
          this.$message.error('网络错误')
          return false
        }
        if (res.status === 502) {
          this.$message.success('编辑书籍类别成功')
          this.getClassList()
          this.editDialogVisible = false
        }
      })
    },
    // 打开添加书籍类别对话框
    addBookClass () {
      this.addForm.classname = ''
      this.addDialogVisible = true
    },
    // 添加书籍类别
    goAddBook () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        var classname = this.addForm.classname
        const { data: res } = await this.$http.post('/addbookclass', { classname: classname })
        if (res.status === 501) {
          this.$message.warning('与已有书籍类别名称相同')
          return false
        }
        if (res.status !== 502) {
          this.$message.error('网络错误')
          return false
        }
        if (res.status === 502) {
          this.$message.success('添加书籍类别成功')
          this.getClassList()
          this.addDialogVisible = false
        }
      })
    },
    // 监听对话框关闭事件
    closedDialog () {
      this.$refs.addFormRef.resetFields()
    }
  }
}
</script>

<style lang="less" scoped>
.el-table {
  width:45%;
}
</style>
