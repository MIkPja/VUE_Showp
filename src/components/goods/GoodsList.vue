<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片内容 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button @click="addGoods" type="primary" id="buttonAdd">添加商品</el-button>
        </el-col>
      </el-row>
      <tree-table
        show-index
        index-text="#"
        :expand-type="false"
        border
        :columns="columns"
        :data="goodsList"
        :selection-type="false"
      ></tree-table>
    </el-card>
    <el-dialog title="添加商品" :visible.sync="setRoleDialogVisible" width="34%">
      <el-form ref="addFormRef" label-width="100px" class="demo-ruleForm">
        <el-form-item label="商品名称">
          <el-input></el-input>
        </el-form-item>
        <el-form-item label="价格">
          <el-input></el-input>
        </el-form-item>
        <el-form-item label="数量">
          <el-input></el-input>
        </el-form-item>
        <el-form-item label="重量">
          <el-input></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="seveSrole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'vueName',
  created () {
    this.getGoodsList()
  },
  data () {
    return {
      setRoleDialogVisible: false,
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      columns: [
        {
          label: '商品ID',
          prop: 'goods_id',
          width: 100
        },
        {
          label: '商品名称',
          prop: 'goods_name'
        },
        {
          label: '操作'
        }
      ],
      goodsList: []
    }
  },
  methods: {
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('请求商品数据失败!')
      }
      this.goodsList = res.data.goods
      console.log(this.goodsList)
    },
    // 添加商品
    async addGoods () {
      this.setRoleDialogVisible = true
    }

    //     const { data: res } = await this.$http.post('goods')
    //       if (res.mate.status !== 201) {
    //         return this.$message.error('添加商品失败!')
    //       }
    //       this.$message.succees('添加商品成功!')
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
</style>
