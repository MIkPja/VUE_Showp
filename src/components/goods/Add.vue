<template>
  <div class>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片内容 -->
    <el-card>
      <!-- <video id="v1" autoplay loop muted>
        <source src="../../assets/1595779127f4105cad066fc923.mp4_last.mp4" type="video/mp4" />
      </video>-->
      <!-- 提示内容 -->
      <el-alert center :closable="false" title="添加商品信息" type="info" effect="dark"></el-alert>
      <!-- 步骤条 -->
      <el-steps align-center :active="active - 0" finish-status="success">
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <el-form
        :model="addForm"
        :rules="addFormrules"
        ref="addFormruleForm"
        label-width="300"
        class="demo-ruleForm"
        label-position="top"
      >
        <el-tabs
          @tab-click="handelTabClick"
          :before-leave="beforeTabLeeave"
          v-model="active"
          tab-position="left"
        >
          <el-tab-pane name="0" label="基本参数">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input type="number" v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input type="number" v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input type="number" v-model="addForm.goods_number"></el-input>
            </el-form-item>

            <el-form-item label="商品分类">
              <el-cascader
                v-model="addForm.goods_cat"
                :options="CateList"
                :props="{
                expandTrigger:'hover',
                  label:'cat_name',
                  value:'cat_id',
                }"
                clearable
                @change="handleCascaderChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane name="1" label="商品参数">
            <!-- 渲染表单的item项 -->
            <el-form-item :label="v.attr_name" v-for="v in manyTableData" :key="v.attr_id">
              <!-- 复选框 -->
              <el-checkbox-group v-model="v.attr_vals">
                <el-checkbox size="mini" :key="i" :label="v1" v-for="(v1,i) in v.attr_vals" border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane name="2" label="商品属性">
            <!-- 渲染表单的item项 -->
            <el-form-item :label="v.attr_name" v-for="v in onlyTableData" :key="v.attr_id">
              <el-input v-model="v.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane name="3" label="商品图片">
            <el-upload
              class="upload-demo"
              :action="uploadUrl"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :on-success="handleSuccess"
              :file-list="fileList"
              list-type="picture"
              :headers="headerObj"
            >
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane name="4" label="商品内容">
            <!-- // 富文本编辑器 -->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <!-- 添加商品按钮 -->
            <el-button id="btn" @click="add" type="primary">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览 -->
    <el-dialog title="图片预览" :visible.sync="PreviewdialogVisible" width="50%">
      <img :src="PreviewPath" alt />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  props: [],
  data () {
    return {
      active: '0',
      value: 6,
      // URL
      uploadUrl: 'http://127.0.0.1:8888/api/private/v1/upload',
      fieList: [],
      // 表单校验规则
      addFormrules: {
        goods_name: [
          { required: true, message: '请输入商品名臣', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品名臣', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品名臣', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品名臣', trigger: 'blur' }
        ]
      },
      // 表单的数据对象
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addForms: {},
      // 图片预览对话框
      PreviewdialogVisible: false,
      CateList: [],
      // 图片预览URL
      PreviewPath: '',
      // 动态参数列表数据
      manyTableData: [],
      // 静态参数列表数据
      onlyTableData: [],
      fileList: [],
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      }
    }
  },
  created () {
    this.getCateList()
    // 测试代码
    // this.active = '2'
    // this.addForm.goods_cat = [1, 3, 6]
    // this.handelTabClick()
  },
  methods: {
    // 监听图片上传成功事件
    handleSuccess (response) {
      //  拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path }
      // 将图片信息对象push 到pics数组中
      this.addForm.pics.push(picInfo)
      console.log(this.addForm.pics)
    },
    // 图片预览事件监听
    handlePreview (file) {
      // 存储预览图片url地址
      this.PreviewPath = file.response.data.url
      // 弹出预览图片对话框
      this.PreviewdialogVisible = true
    },
    // 图片删除监听事件
    handleRemove (file) {
      // 当前移除图片的临时路径
      const filePath = file.response.data.tmp_path
      const x = this.addForm.pics.findIndex((i) => i.pic === filePath)
      this.addForm.pics.splice(x, 1)
    },
    // 级联选择框选项变化触发函数
    handleCascaderChange () {
      if (this.addForm.goods_cat.length < 3) {
        this.addForm.goods_cat = []
      }
    },
    // 获取数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      // 存储数据
      this.CateList = res.data
    },
    // from表单切换校验
    beforeTabLeeave (New, Old) {
      // 通过:before-leave事件 监听tabs的切换得到参数
      // New == 即将去往的页面
      // Old == 即将离开的页面
      if (this.addForm.goods_cat.length !== 3 && Old === '0') {
        this.$message({
          message: '未选择商品分类',
          type: 'error',
          showClose: true
        })
        return false
      }
    },
    // 监听tab切换
    async handelTabClick () {
      if (this.active === '1') {
        if (this.cateId === null || this.cateId === undefined) return
        // tab切换触发事件
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'many' }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取参数列表失败！')
        }
        res.data.forEach((v) => {
          v.attr_vals = v.attr_vals ? v.attr_vals.split(' ') : []
        })
        this.manyTableData = res.data
      } else if (this.active === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'only' }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取参数列表失败！')
        }
        this.onlyTableData = res.data
      }
    },
    add () {
      // 表单预校验
      this.$refs.addFormruleForm.validate(async (isValid) => {
        if (!isValid) {
          // // 通过表单验证发起请求添加用户
          // const { data: res } = await this.$http.post('goods', this.addForm)
          // debugger
          this.$message({
            message: '请填写完整商品信息',
            type: 'error',
            showClose: true
          })
        } else {
          // this.addForms.goods_cat =
          // this.addForms.goods_cat.join(',')
          const form = _.cloneDeep(this.addForm)
          form.goods_cat = form.goods_cat.join(',')
          // 处理动态参数
          this.manyTableData.forEach(v => {
            const newInfo = {
              attr_id: v.attr_id,
              attr_value: v.attr_vals
            }
            this.addForm.attrs.push(newInfo)
          })
          // 处理静态属性
          this.onlyTableData.forEach(v => {
            const newInfo = {
              attr_id: v.attr_id,
              attr_value: v.attr_vals
            }
            this.addForm.attrs.push(newInfo)
          })
          // 赋值提交
          form.attrs = this.addForm.attrs
          // 发起请求添加商品
          // 商品名必须是唯一的
          const { data: res } = await this.$http.post('goods', form)
          // debugger
          if (res.meta.status !== 201) {
            this.$message({
              message: '添加商品失败!',
              type: 'error',
              showClose: true
            })
          } else {
          // 成功添加
            this.$message({
              message: '添加商品成功',
              type: 'success',
              showClose: true
            })
          }
        }
      })
    }
  },
  computed: {
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      } else {
        return null
      }
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
#btn {
  margin-bottom: 15px;
}
.el-pagination {
  margin-top: 15px;
}
.el-alert {
  margin-bottom: 15px;
}
.block {
  height: 400px;
}
.el-tabs {
  margin-top: 15px;
}
/deep/ input {
  width: 300px;
}
/deep/ .el-form-item__label {
  font-size: 18px;
  font-weight: 800;
  text-indent: 1em;
}
/deep/ .el-checkbox-group {
  padding-left: 15px;
}
video {
  position: fixed;
  right: 0px;
  bottom: 0px;
  min-width: 100%;
  min-height: 100%;
  height: auto;
  width: auto;
  /*加滤镜*/
  /*filter: blur(15px); //背景模糊设置 */
  /*-webkit-filter: grayscale(100%);*/
  /*filter:grayscale(100%); //背景灰度设置*/
  z-index: -11;
}
source {
  min-width: 100%;
  min-height: 100%;
  height: auto;
  width: auto;
}

/deep/ .el-breadcrumb__inner {
  color: rgb(253, 253, 253);
}
.el-main[data-v-957c9522] {
  background-color: rgba(0, 0, 0, 0);
  padding-top: 100px;
}
span[data-v-8de88330] {
  color: rgb(253, 253, 253);
}
/deep/ .el-upload-list__item {
  width: 300px;
}
img {
  width: 100%;
}
/deep/ .ql-container {
  min-height: 300px;
}
#btn {
  margin-top: 15px;
}
</style>
