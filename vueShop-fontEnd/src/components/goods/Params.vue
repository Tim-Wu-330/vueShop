<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert show-icon title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false"></el-alert>
      <el-row class="cateSelection">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="catelist"
            @change="handleChange"
            :props="cateProps"
          ></el-cascader>
        </el-col>
      </el-row>

      <!-- tab页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">
            添加参数
          </el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand"></el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">
                  编辑
                </el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">
                  删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 添加静态参数的面板 -->
        <el-tab-pane label="静态参数" name="only">
          <!-- 添加属性的按钮 -->
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">
            添加参数
          </el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand"></el-table-column>
            <!-- 索引行 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)"
                  >编辑
                </el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)"
                  >删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 添加参数对话框 -->
        <el-dialog :title="'添加' + titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
          <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
            <el-form-item :label="titleText" prop="attr_name">
              <el-input v-model="addForm.attr_name"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addParams">确 定</el-button>
          </span>
        </el-dialog>

        <!-- 修改参数对话框 -->
        <el-dialog :title="'修改' + titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
          <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
            <el-form-item :label="titleText" prop="attr_name">
              <el-input v-model="editForm.attr_name"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editParams">确 定</el-button>
          </span>
        </el-dialog>
      </el-tabs>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类列表
      catelist: [],
      // 级联选择框的配置对象
      cateProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
        expandTrigger: "hover",
      },
      // 级联选择框双向绑定到的数组
      selectedCateKeys: [],
      //   被激活的页签名称
      activeName: "many",
      //   动态参数数据
      manyTableData: [],
      //   控制添加对话框的显示与隐藏
      addDialogVisible: false,
      // 静态参数数据
      onlyTableData: [],
      // 添加参数的表单数据对象
      addForm: {
        attr_name: "",
      },
      // 添加表单的验证规则对象
      addFormRules: {
        attr_name: [{ required: true, message: "请输入参数名称", trigger: "blur" }],
      },
      //   控制修改对话框的显示与隐藏
      editDialogVisible: false,
      // 修改的表单数据对象
      editForm: {},
      // 修改表单的验证规则对象
      editFormRules: {
        attr_name: [{ required: true, message: "请输入参数名称", trigger: "blur" }],
      },
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //获取所有的商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get("categories")
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.catelist = res.data
      console.log(this.catelist)
    },
    // 级联选择框选中项变化，会触发这个函数
    handleChange() {
      this.getParamsData()
    },
    // tab 页签点击事件的处理函数
    handleTabClick() {
      console.log(this.activeName)
      this.getParamsData()
    },
    // 获取参数的列表数据
    async getParamsData() {
      // 证明选中的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        return
      }
      // 证明选中的是三级分类
      console.log(this.selectedCateKeys)
      console.log(this.cateId)
      // 根据所选的分类Id和当前所处的面板，获取对应参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: { sel: this.activeName },
      })
      if (res.meta.status !== 200) {
        return this.$message.error("获取参数列表失败！")
      }
      console.log(res.data)
      if (this.activeName === "many") {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 监听添加对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },

    // 点击按钮，添加参数
    addParams() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName,
        })
        if (res.meta.status !== 201) {
          return this.$message.error("添加参数失败！")
        }
        this.$message.success("添加参数成功！")
        this.addDialogVisible = false
        this.getParamsData()
      })
    },

    async showEditDialog(attr_id) {
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`, {
        params: { attr_sel: this.activeName },
      })

      if (res.meta.status !== 200) {
        return this.$message.error("获取参数信息失败！")
      }

      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 点击按钮，修改参数信息
    editParams() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName,
        })

        if (res.meta.status !== 200) {
          return this.$message.error("修改参数失败！")
        }

        this.$message.success("修改参数成功！")
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 重置修改的表单
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 根据Id删除对应的参数项
    async removeParams(attr_id) {
      const confirmResult = await this.$confirm("此操作将永久删除该参数, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).catch((err) => err)

      // 用户取消了删除的操作
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除！")
      }

      // 删除的业务逻辑
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)

      if (res.meta.status !== 200) {
        return this.$message.error("删除参数失败！")
      }

      this.$message.success("删除参数成功！")
      this.getParamsData()
    },
  },
  computed: {
    //   如果按钮需要被禁用，则返回true，否则返回false
    isBtnDisabled() {
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类Id
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    titleText() {
      if (this.activeName === "many") {
        return "动态参数"
      } else {
        return "静态参数"
      }
    },
  },
}
</script>

<style lang="less" scoped>
.cateSelection {
  margin: 15px 0;
}
</style>
