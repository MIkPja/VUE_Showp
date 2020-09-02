<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片布局 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button @click="addCate_dialog" type="primary" id="buttonAdd">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 标题内容 -->
      <tree-table
        :show-row-hover="false"
        border
        show-index
        index-text="#"
        :expand-type="false"
        :selection-type="false"
        :data="cateList"
        :columns="columns"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            style="color:lightgreen"
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
          ></i>
          <i style="color:red" class="el-icon-edit" v-else></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">普通</el-tag>
          <el-tag size="mini" v-else-if="scope.row.cat_level === 1" type="success">VIP</el-tag>
          <el-tag size="mini" v-else type="warning">SVIP</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope>
          <el-button @click="editCate" type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="userInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="userInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 添加分类弹出框 -->
    <el-dialog
      @closed="addCateDialogClosed"
      title="添加分类"
      :visible.sync="addRoleDialogVisible"
      width="34%"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类" prop="cat_name">
          <!-- 级联选择 -->
          <div class="block">
            <!-- options 指定数据源 -->
            <!-- props 配置选项 -->
            <el-cascader
              expandTrigger="hover"
              v-model="selectedParentKeys"
              :options="ParrentCateList"
              :props="cascaderProps"
              @change="handleCascaderChange"
              clearable
              :checkStrictly="true"
            ></el-cascader>
          </div>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑分类弹出框 -->
    <!-- <el-dialog title="编辑分类" :visible.sync="editRoleDialogVisible" width="34%">
      <el-form
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类" prop="cat_name"></el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>-->
  </div>
</template>

<script>
export default {
  props: [],
  data () {
    return {
      userInfo: {
        type: 3, // 分页
        pagenum: 1, // 当前页
        pagesize: 5 // 每页多少条
      },
      columns: [
        {
          label: '商品分类',
          prop: 'cat_name'
        },
        // template
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      addRoleDialogVisible: false,
      editRoleDialogVisible: false,
      cateList: [],
      // 父级分类列表
      ParrentCateList: [],
      // 级联选择框获取到的ID
      selectedParentKeys: [],
      total: 0,
      // 级联分类数据
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      addCateForm: {
        cat_name: '',
        cat_pid: '',
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    // 生命周期函数调用获取数据方法
    this.getCateList()
  },
  methods: {
    // 获取数据方法
    async getCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: this.userInfo
      })
      // 存储请求到的数据
      this.cateList = res.data.result
      // 存储请求到的total值 (总条数)
      this.total = res.data.total
    },
    handleSizeChange (newSize) {
      this.userInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      this.userInfo.pagenum = newPage
      this.getCateList()
    },
    // 添加分类
    addCate_dialog () {
      // 获取父级分类列表
      this.getParrentCateList()
      this.addRoleDialogVisible = true
    },
    async getParrentCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据分类失败')
      }
      // 存储父级分类列表数据
      this.ParrentCateList = res.data
    },
    // 编辑分类处理函数
    editCate () {
      this.editRoleDialogVisible = true
    },
    // 添加分类处理函数
    async addCate () {
      // validate 可以返回表单通过验证与否的布尔值
      const { data: res } = await this.$http.post('categories', this.addCateForm)
      // console.log(res)
      if (res.meta.status === 201) {
        // 成功
        this.$message({
          message: '添加分类成功',
          type: 'success',
          showClose: true
        })
        this.addRoleDialogVisible = false
        this.getCateList()
        console.log(1)
      } else {
        // 失败
        this.$message({
          message: '添加分类失败',
          type: 'error',
          showClose: true
        })
      }
    },
    // 级联选择框
    handleCascaderChange () {
      console.log(this.selectedParentKeys)
      if (this.selectedParentKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedParentKeys[
          this.selectedParentKeys.length - 1
        ]
        this.addCateForm.cat_level = this.selectedParentKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
      console.log(this.addCateForm.cat_pid)
      console.log(this.addCateForm.cat_level)
    },
    addCateDialogClosed () {
      // 重置表单数据
      this.$refs.addCateFormRef.resetFields()
      // 清空select 数据
      this.selectedParentKeys = []
      // 清空级联已选择的数据
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    }
  }
}
</script>

<style scoped lang = 'less'>
#buttonAdd {
  margin-bottom: 15px;
}
.zk-table {
  border-radius: 5px !important;
}
.el-pagination {
  margin-top: 10px !important;
}
.el-cascader {
  width: 100%;
}
</style>
