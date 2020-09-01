<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>校色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button @click="addUser" type="primary" id="buttonAdd">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <el-table-column label type="expand">
          <template slot-scope="scope">
            <el-row
              class="bdtop vcenter"
              :class="{
              'bdbottom': i1 === scope.row.children.length -1
            }"
              v-for="(v1,i1) in scope.row.children"
              :key="v1.id"
            >
              <el-col :span="5">
                <el-tag>{{v1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row
                  class="vcenter"
                  :class="{'bdtop': i2 !== 0}"
                  v-for="(v2,i2) in v1.children"
                  :key="v2.id"
                >
                  <el-col :span="6">
                    <el-tag type="success">{{v2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      closable
                      @close="removeRightId(scope.row,v3.id)"
                      type="warning"
                      v-for="(v3) in v2.children"
                      :key="v3.id"
                    >{{v3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="400">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            <el-button
              @click="showSerRightDialog(scope.row)"
              size="mini"
              type="warning"
              icon="el-icon-setting"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 弹出层 -->
    <el-dialog
      @close="setRightDialogClosed"
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="25%"
    >
      <!-- 输入框 -->
      <el-tree
        ref="treeRef"
        :data="rightsList"
        :default-checked-keys="defKeys"
        show-checkbox
        node-key="id"
        default-expand-all
        :props="treeProps"
      ></el-tree>
      <el-form width="25%"></el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRight">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolelist: [],
      rightsList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的节点ID值
      defKeys: [],
      roleId: undefined,
      setRightDialogVisible: false
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败')
      }
      this.rolelist = res.data
    },
    async showSerRightDialog (row) {
      this.roleId = row.id
      const { data: res } = await this.$http.get('rights/tree')
      this.rightsList = res.data
      this.getLeafKeys(row)
      // 弹出层显示
      this.setRightDialogVisible = true
    },
    getLeafKeys (item) {
      if (!item.children) {
        this.defKeys.push(item.id)
      } else {
        item.children.forEach((subItem) => {
          this.getLeafKeys(subItem)
        })
      }
    },
    // 监听弹出框关闭事件  清空权限列表中的数据
    setRightDialogClosed () {
      this.defKeys = []
    },
    addUser () {},
    // 根据ID删除对应的权限
    async removeRightId (role, rightId) {
      const fonfirmResult = await this.$confirm(
        '此操作将永久删除权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      if (fonfirmResult !== 'cancel') {
        const { data: res } = await this.$http.delete(
          `roles/${role.id}/rights/${rightId}`
        )
        this.$message.success('删除成功')
        role.children = res.data
      }
    },
    async setRight () {
      const rids = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      await this.$http.post('roles/' + this.roleId + '/rights', {
        rids: rids.join(',')
      })
      console.log(rids)

      this.$message({
        message: '分配成功',
        type: 'success',
        showClose: true
      })
      this.setRightDialogVisible = false
      this.getRolesList()
    }
  }
}
</script>

<style scoped >
#buttonAdd {
  margin-bottom: 15px;
}
.bdtop {
  border-top: 1px solid #00000061;
}
.bdbottom {
  border-bottom: 1px solid #00000061;
}
.el-tag {
  margin: 7px;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
