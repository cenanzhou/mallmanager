<template>
  <el-card class="box-card">
    <!-- 1、面包屑 -->
    <!-- 首页/用户管理/用户列表 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 2、搜索 -->
    <el-row class="searchRow">
      <el-col>
        <el-input
          @clear="loadUserList()"
          clearable
          placeholder="请输入内容"
          v-model="query"
          class="inputSearen"
        >
          <el-button
            @click="searchUser()"
            slot="append"
            icon="el-icon-search"
          ></el-button>
        </el-input>
        <el-button type="success" @click="showAddUserDia()">添加用户</el-button>
      </el-col>
    </el-row>

    <!-- 3、表格 -->
    <el-table height="300px" :data="userlist" style="width: 100%">
      <el-table-column type="index" label="#" width="60"> </el-table-column>
      <el-table-column prop="username" label="姓名" width="80">
      </el-table-column>
      <el-table-column prop="email" label="邮箱"> </el-table-column>
      <el-table-column prop="mobile" label="电话"> </el-table-column>

      <el-table-column label="创建时间">
        <!-- 如果单元格内显示的内容不是字符串(文本)，
        需要给被显示的内容外层包裹一个template-->
        <!-- template内部要用数据 设置slot-scope属性
            该属性的值是要用数据create_time的数据源userlist
            userlist.row->数组中的每个对象
        -->
        <template slot-scope="userlist">
          {{ userlist.row.create_time | fmtdate }}
        </template>
      </el-table-column>

      <el-table-column label="用户状态">
        <!--template  slot-scope -->
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
          >
          </el-switch>
        </template>
      </el-table-column>

      <el-table-column prop="address" label="操作">
        <template slot-scope="scope">
          <el-button
            size="mini"
            plain
            type="primary"
            icon="el-icon-edit"
            circle
            @click="showEditUserDia(scope.row)"
          ></el-button>
          <el-button
            size="mini"
            plain
            type="success"
            icon="el-icon-check"
            circle
          ></el-button>
          <el-button
            size="mini"
            plain
            type="danger"
            icon="el-icon-delete"
            circle
            @click="showDeleUserMsgBox(scope.row.id)"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 4、分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="2"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    >
    </el-pagination>

    <!-- 对话框 -->
    <!-- 添加用户的对话框 -->
    <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd">
      <el-form :model="form">
        <el-form-item label="用户名" label-width="100px">
          <el-input v-model="form.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" label-width="100px">
          <el-input v-model="form.password" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" label-width="100px">
          <el-input v-model="form.email" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="电话" label-width="100px">
          <el-input v-model="form.mobile" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 编辑用户的对话框 -->
    <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit">
      <el-form :model="form">
        <el-form-item label="用户名" label-width="100px">
          <el-input disabled v-model="form.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" label-width="100px">
          <el-input v-model="form.email" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="电话" label-width="100px">
          <el-input v-model="form.mobile" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleEdit = false">取 消</el-button>
        <el-button type="primary" @click="dialogFormVisibleEdit = false">确 定</el-button>
      </div>
    </el-dialog>
  </el-card>
</template>
<script>
export default {
  data () {
    return {
      query: '',
      // 表格绑定的数据
      userlist: [],
      // 分页相关的数据
      total: -1,
      pagenum: 1,
      pagesize: 2,
      // 添加对话的属性
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      // 添加对话的属性
      form: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      }
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    // 编辑用户 - 显示对话框
    showEditUserDia (user) {
      this.form = user
      // 获取用户信息
      this.dialogFormVisibleEdit = true
    },
    // 删除用户 - 打开消息盒子(config)
    showDeleUserMsgBox (userId) {
      this.$confirm('删除用户?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          // 发送删除的请求
          // 1、data中找到userId
          // 2、把userId以参数形式传进来
          const res = await this.$http.delete(`users/${userId}`)
          console.log(res)
          if (res.data.meta.status === 200) {
            this.pagenum = 1
            // 更新视图
            this.getUserList()
            this.$message({
              type: 'success',
              message: res.data.meta.msg
            })
          }
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 添加用户 - 发送请求
    async addUser () {
      // 2、关闭对话框
      this.dialogFormVisibleAdd = false
      const res = await this.$http.post(`users`, this.form)
      console.log(res)
      const {
        meta: { status, msg }
        // data,
      } = res.data
      if (status === 201) {
        // 1、提示成功
        this.$message.success(msg)
        // 3、更新视图
        this.getUserList()
        // 4、清空文本框
        this.form = {}
      } else {
        this.$message.warning(msg)
      }
    },
    // 添加用户
    showAddUserDia () {
      this.dialogFormVisibleAdd = true
    },
    // 清空搜索框，重新获取数据
    loadUserList () {
      this.getUserList()
    },
    // 搜索用户
    searchUser () {
      // 按照input绑定的query
      this.getUserList()
    },
    // 分页相关的方法
    handleSizeChange (val) {
      console.log(`每页 ${val} 条`)
      this.pagesize = val
      // 回到第一页
      this.pagenum = 1
      this.getUserList()
    },
    handleCurrentChange (val) {
      console.log(`当前页: ${val}`)
      this.pagenum = val
      this.getUserList()
    },

    // 获取用户列表的请求
    async getUserList () {
      // 需要授权
      // 请求头
      const AUTH_TOKE = localStorage.getItem('token')
      this.$http.defaults.headers.common['Authorization'] = AUTH_TOKE
      const res = await this.$http.get(
        `users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}`
      )
      console.log(res)
      const {
        meta: { status, msg },
        data: { users, total }
      } = res.data
      if (status === 200) {
        // 1、给表格数据赋值
        this.userlist = users
        // 2、给total赋值
        this.total = total
        // 3、提示
        this.$message.success(msg)
      } else {
        // 提示
        this.$message.warning(msg)
      }
    }
  }
}
</script>
<style>
.box-card {
  height: 100%;
}
.inputSearen {
  width: 300px;
}
.searchRow {
  margin-top: 20px;
}
</style>
