<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
    <el-button type="primary">添加分类</el-button>

    <!-- 表格区域 -->
    <tree-table :data="catilist" :columns="columns" :selection-type='false' :expand-type="false" show-index index-text="#" border :show-row-hover="false">

    </tree-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 请求数据携带参数
      queryInfo: {
        type: '', // 请求几层数据
        pagenum: 5, // 当前页显示几条数据
        pagesize: 1// 显示第几页数据
      },
      // 表单列表数据
      catilist: [],
      // 商品总列数
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        }
      ],
      // 商品总页数
      total: ''
    }
  },
  // 函数内容
  methods: {
    // 获得商品分类数据
    async getgories () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取数据失败')
      console.log(res)
      this.$message.success('获取数据成功')
      this.catilist = res.data.result
      this.total = res.data.total
      // console.log(res.data.result)
    }
  },
  // 钩子函数
  created () {
    this.getgories()
  },
  // 计算属性
  computend () {}
}
</script>

<style scoped lang = 'less'>
</style>
