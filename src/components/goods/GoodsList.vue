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
      <!-- 内容 -->
      <el-row id="btn" :gutter="20">
        <el-col :span="7">
          <el-input clearable @clear="getGoodsList" v-model="queryInfo.query" placeholder="请输入内容">
            <el-button
              @click="getGoodsList"
              v-model="queryInfo.query"
              slot="append"
              icon="el-icon-search"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddGoods">添加商品</el-button>
        </el-col>
      </el-row>
      <tree-table
        stripe
        show-index
        index-text="#"
        :expand-type="false"
        border
        :columns="columns"
        :data="goodsList"
        :selection-type="false"
      >
        <!-- 创建时间 -->
        <template slot="time" slot-scope="scope">{{scope.row.add_time | dataFormat}}</template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button size="mini" type="danger" icon="el-icon-edit">编辑</el-button>
          <el-button
            @click="removeGoodsById(scope.row.goods_id)"
            size="mini"
            type="primary"
            icon="el-icon-delete"
          >删除</el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 5, 10, 200]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <el-dialog
      @closed="addGoodsDialogClosed"
      title="添加商品"
      :visible.sync="addRoleDialogVisible"
      width="34%"
    >
      <el-form :model="addFrom" ref="addFormRef" label-width="100px" class="demo-ruleForm">
        <el-form-item label="商品名称">
          <el-input v-model="addFrom.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="价格">
          <el-input v-model="addFrom.goods_price"></el-input>
        </el-form-item>
        <el-form-item label="数量">
          <el-input v-model="addFrom.goods_number"></el-input>
        </el-form-item>
        <el-form-item label="重量">
          <el-input v-model="addFrom.goods_weight"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'vueName',
  created () {
    // 获取商品列表信息
    this.getGoodsList()
  },
  data () {
    return {
      // 控制弹出层
      addRoleDialogVisible: false,
      // 渲染表格数据
      queryInfo: {
        query: '', // 关键字
        pagenum: 1, // 当前第几页
        pagesize: 10 // 每页显示多少条
      },
      // 表单数据
      addFrom: {
        goods_name: '',
        goods_price: '',
        goods_number: '',
        goods_weight: ''
      },
      // 表格数据
      columns: [
        {
          label: '商品名称',
          prop: 'goods_name',
          width: 1000
        },
        {
          label: '商品价格',
          prop: 'goods_price',
          width: 100
        },
        {
          label: '商品重量',
          prop: 'goods_weight',
          width: 100
        },
        {
          label: '创建时间',
          type: 'template',
          template: 'time',
          width: 200
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      goodsList: [],
      total: 0
    }
  },
  methods: {
    // 根据ID删除对应的商品信息
    async removeGoodsById (id) {
      // 提示用户是否删除
      const confirm = await this.$confirm(
        '此操作将永久删除该商品, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      )
      if (confirm === 'confirm') {
        // 发送请求
        const { data: res } = await this.$http.delete('goods/' + id)
        // 判断删除是否成功
        if (res.meta.status !== 200) {
          return this.$message.error('删除用户信息失败')
        }
        // 删除成功
        this.$message.success('删除用户信息成功')
        // 删除成功后重新渲染用户列表
        this.getGoodsList()
      }
    },
    // 获取商品列表信息
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('请求商品数据失败!')
      }
      this.goodsList = res.data.goods
      this.total = res.data.total
      console.log(this.goodsList)
    },
    // 添加商品
    addGoods () {
      this.addRoleDialogVisible = true
    },
    // 监听弹出框关闭事件
    addGoodsDialogClosed () {
      this.$refs.addFormRef.resetFields()
      this.addFrom = {}
    },
    // 分页
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    goAddGoods () {
      this.$router.push('/goods/Add')
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
#btn {
  margin-bottom: 15px;
}
.el-pagination {
  margin-top: 15px;
}
</style>
