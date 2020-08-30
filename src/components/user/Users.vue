<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <!-- 内容 -->
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input v-model="queryInfo.query" placeholder="请输入内容">
            <el-button @click="getUserList" slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="userlist" border stripe>
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="姓名" width="180" prop="username"></el-table-column>
        <el-table-column label="邮箱" width="180" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态" prop="mg_state" width="110">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              active-color="#13ce66"
              inactive-color="#ff4949"
              @change="userStateChange(scope.row)"
            ></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary "
              icon="el-icon-edit"
              @click="eeditDialogVisible(scope.row.id)"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button @click="removeUserById(scope.row.id)" size="mini" type="danger" icon="el-icon-delete"></el-button>
            <el-tooltip
              class="item"
              effect="dark"
              :enterable="false"
              content="分配角色"
              placement="top"
            >
              <el-button size="mini" type="warning" icon="el-icon-setting"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <div class="block">
        <span class="demonstration"></span>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[1, 2, 5, 10]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
        ></el-pagination>
      </div>
    </el-card>
    <el-dialog
      title="添加用户"
      @closed="hanldeAddDialogClose"
      :visible.sync="addDialogVisible"
      width="25%"
    >
      <!-- 输入框 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item class="boxNow" label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <!-- 邮箱 -->
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <!-- 手机 -->
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click.stop="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑用户 -->
    <el-dialog
      title="编辑用户"
      @closed="hanldeEditDialogClose"
      :visible.sync="editDialogVisible"
      width="25%"
    >
      <!-- 输入框 -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        @closed="hanldeAddDialogClose"
        ref="editFormRef"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item class="boxNow" label="用户名">
          <el-input class="input-username" v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <!-- 邮箱 -->
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <!-- 手机 -->
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click.stop="editUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    // 验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/

      if (regEmail.test(value)) {
        // 合法的邮箱
        return cb()
      }
      cb(new Error('请输入合法的邮箱'))
    }

    // 验证手机号的规则
    var checkMobile = (rule, value, cb) => {
      // 验证手机号的正则表达式
      const regMobile = /^(0|86|17951)?(19[0-9]|13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/

      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的手机号'))
    }
    return {
      // 获取用户列表参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      userlist: [],
      total: 0,
      // 添加用户数据
      addDialogVisible: false,
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '长度在 2 到 10 个字符',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '长度在 6 到 15 个字符',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      addForm: {
        username: '张三',
        password: '*******',
        email: 'al1345678@lookout.com',
        mobile: '13894665322'
      },
      //  编辑用户数据
      editDialogVisible: false,
      editFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '长度在 2 到 10 个字符',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '长度在 6 到 15 个字符',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      editForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      }
    }
  },
  created () {
    // 初始化渲染用户列表
    this.getUserList()
  },
  methods: {
    // 用户数据列表渲染
    async getUserList () {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('用户数据获取失败')
      }
      // 存储数据
      this.userlist = res.data.users
      this.total = res.data.total
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 用户状态函数
    async userStateChange (userInfo) {
      const { data: res } = await this.$http.put(
        `users/${userInfo.id}/state/${userInfo.mg_state}`
      )
      if (res.status === 200) {
        return this.$message({
          message: '修改用户状态失败!',
          type: 'success',
          showClose: false
        })
      }
      this.$message({
        message: '修改用户状态成功!',
        type: 'success',
        showClose: true
      })
    },
    // 重置表单数据
    hanldeAddDialogClose () {
      this.$refs.addFormRef.resetFields()
    },
    // 添加用户
    addUser () {
      // 表单预校验
      this.$refs.addFormRef.validate(async (isValid) => {
        if (!isValid) return
        // 通过表单验证发起请求添加用户
        const { data: res } = await this.$http.post('users', this.addForm)
        // debugger
        if (res.meta.status !== 201) {
          this.$message({
            message: res.meta.msg,
            type: 'success',
            showClose: true
          })
        } else {
          // 成功添加
          this.$message({
            message: res.meta.msg,
            type: 'success',
            showClose: true
          })
          // 重新渲染用户数据
          this.getUserList()
          // 关闭对话框
          console.log(this)
          this.addDialogVisible = false
        }
      })
    },
    // 重置表单数据
    hanldeEditDialogClose () {
      this.$refs.editFormRef.resetFields()
    },

    async eeditDialogVisible (id) {
      const { data: res } = await this.$http.get('users' + '/' + id)
      if (res.meta.status !== 200) {
        return this.$message({
          message: res.meta.msg,
          type: 'success',
          showClose: true
        })
      } else {
        this.editForm = res.data
        this.editDialogVisible = true
      }
    },
    editUser () {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message({
            message: '输入有误',
            type: 'warning',
            showClose: true
          })
        }
        const { data: res } = await this.$http.put(
          'users/' + this.editForm.id,
          // 接受数据是对象
          { email: this.editForm.email, mobile: this.editForm.mobile }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('更新用户信息失败')
        }
        this.editDialogVisible = false
        this.getUserList()
        this.$message.success('更新用户信息成功')
      })
    },
    // 根据ID删除对应的用户信息
    async removeUserById (id) {
      // 提示用户是否删除
      const confirm = await this.$confirm('此操作将永久删除该用户信息, 是否继续?', '提示', {
        confirmButtonText: '确定删除',
        cancelButtonText: '取消操作',
        type: 'warning'
      })
      if (confirm === 'confirm') {
        // 发送请求
        const { data: res } = await this.$http.delete('roles/' + id)
        // 判断删除是否成功
        if (res.meta.status !== 200) {
          return this.$message.error('删除用户信息失败')
        }
        // 删除成功
        this.$message.success('删除用户信息成功')
        // 删除成功后重新渲染用户列表
        this.getUserList()
      }
    }
  }
}
</script>

<style scoped lang = 'less'>
.el-pagination {
  margin-top: 15px;
}
.text {
  font-size: 14px;
}

.item {
  margin-bottom: 18px;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.item[data-v-0d4166b4] {
  margin-bottom: 0 !important;
}
.clearfix:after {
  clear: both;
}

.el-breadcrumb {
  margin-bottom: 15px;
}

.el-table {
  margin-top: 15px;
  font-size: 12px;
}
.el-input__inner {
  user-select: none;
}
</style>
