<template>
  <!-- 面包屑导航 -->
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-table :data="rightList"  border stripe>
         <el-table-column type="index"></el-table-column>
        <el-table-column label="权限名称" prop="authName"></el-table-column>
        <el-table-column label="路径" prop="path"></el-table-column>
        <el-table-column label="权限等级" prop="level">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.level === '0'">一级</el-tag>
            <el-tag type="success" v-else-if="scope.row.level === '1'">二级</el-tag>
            <el-tag type="warning" v-else>三级</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
  <!-- 主要内容 -->
</template>

<script>
export default {
  props: [],
  data () {
    return {
      rightList: []
    }
  },
  created () {
    this.getRightList()
  },
  methods: {
    // 用户数据列表渲染
    async getRightList () {
      const { data: res } = await this.$http.get('rights/list')
      this.rightList = res.data
      console.log(this.rightList)
      console.log(res)
    }
  }
}
</script>

<style scoped lang = 'less'>
.el-table {
  box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.15) !important;
  border: 1px solid #cccccc !important;
  border-radius: 5px;
}
.el-tag.el-tag--warning {
    background-color: hsl(66, 100%, 50%);
    border-color: hsl(60, 13%, 95%);
    color: red;
}
</style>
