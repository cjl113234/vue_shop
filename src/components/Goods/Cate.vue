<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary"
                     @click="showAddCateDialog()">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格区域 -->
      <tree-table :data="cateList"
                  :columns="columns"
                  :selection-type="false"
                  :expand-type="false"
                  show-index
                  index-text='#'
                  border
                  class="treeTable">
        <!-- 是否有效 -->
        <template slot="isOk"
                  slot-scope="scope">
          <i class="el-icon-success"
             v-if="scope.row.cat_deleted === false"
             style="color:lightgreen"></i>
          <i class="el-icon-error"
             v-else
             style="color:red"></i>
        </template>

        <!-- 排序 -->
        <template slot="order"
                  slot-scope="scope">
          <el-tag size="mini"
                  v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini"
                  type="success"
                  v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag size="mini"
                  type="danger"
                  v-else>三级</el-tag>
        </template>

        <!-- 操作 -->
        <template slot="opt"
                  slot-scope="scope">
          <el-button type="primary"
                     icon="el-icon-edit"
                     size="mini"
                     @click="showEditCateDialog(scope.row.cat_id)">编辑</el-button>
          <el-button type="danger"
                     icon="el-icon-delete"
                     size="mini"
                     @click="removeCateById(scope.row.cat_id)">删除</el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[3, 5, 10, 15]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类"
               :visible.sync="addCateDialogVisible"
               width="50%"
               @close="addCateDialogClosed()">
      <!-- 添加分类的表单 -->
      <el-form :model="addCateRuleForm"
               :rules="addCateRuleFormRules"
               ref="addCateRuleFormRulesForm"
               label-width="100px">
        <el-form-item label="分类名称："
                      prop="cat_name">
          <el-input v-model="addCateRuleForm.cat_name"></el-input>
        </el-form-item>

        <el-form-item label="父级分类：">
          <!-- options用来指定数据源 -->
          <!-- props用来指定配置对象 -->
          <el-cascader v-model="selectedKeys"
                       :options="parentCateList"
                       :props="{ expandTrigger: 'hover',value: 'cat_id',
                                label: 'cat_name',
                                children: 'children' }"
                       @change="parentCateHandleChange()"
                       clearable
                       change-on-select></el-cascader>
        </el-form-item>
      </el-form>

      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addCate()">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑商品分类的对话框 -->
    <el-dialog title="修改商品分类"
               :visible.sync="editCateDialogVisible"
               width="50%"
               @close="editCateDialogClosed()">
      <el-form :model="editCateForm"
               :rules="editCateFormRules"
               ref="editCateFormRef"
               label-width="100px">
        <el-form-item label="商品名称"
                      prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editCateInfo()">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      // 查询条件
      queryInfo: {
        type: 3, pagenum: 1, pagesize: 5
      },
      // 商品分类的数据列表，默认为空
      cateList: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用的模板名称
          template: 'isOk'
        },
        {
          label: '排序',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用的模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用的模板名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 控制修改分类名称对话框的显示与隐藏
      editCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateRuleForm: {
        // 将要添加的分类的名称
        cat_name: '',
        // 父级分类的ID，默认要添加的是一级分类
        cat_pid: 0,
        // 分类的等级，默认要添加的是一级分类
        cat_level: 0
      },
      // 添加分类表单的验证规则对象
      addCateRuleFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 选中的父级分类的ID数组
      selectedKeys: [],

      editCateForm: {
        // 将要修改的分类名称的名称
        cat_name: '',
        // 父级分类的ID，默认要修改的是一级分类
        cat_pid: 0,
        // 分类的等级，默认要修改的是一级分类
        cat_level: 0
      },
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' },
          { min: 1, max: 10, message: '商品名称的长度在1-10个字符之间', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      // console.log(res.data)
      // 把数据列表赋值给cateList
      this.cateList = res.data.result
      // 为总数据条数赋值
      this.total = res.data.total
    },
    // 监听pagesize改变的事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum改变的事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮，展示添加分类的对话框
    showAddCateDialog () {
      // 先获取父级分类的数据列表，再展示对话框
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败！')
      }
      // console.log(res.data)
      this.parentCateList = res.data
    },
    // 选择项发生变化，触发这个函数
    parentCateHandleChange () {
      // console.log(this.selectedKeys)
      // 如果selectKeys数组中的length大于0，证明选中了父级分类
      // 反之，就说明没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的ID
        this.addCateRuleForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 为当前分类的等级赋值
        this.addCateRuleForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的ID
        this.addCateRuleForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateRuleForm.cat_level = 0
      }
    },
    // 点击按钮，添加新的分类
    addCate () {
      // console.log(this.addCateRuleForm)
      this.$refs.addCateRuleFormRulesForm.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateRuleForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听对话框的关闭事件，重置表单数据
    addCateDialogClosed () {
      this.$refs.addCateRuleFormRulesForm.resetFields()
      this.selectedKeys = []
      this.addCateRuleForm.cat_level = 0
      this.addCateRuleForm.cat_pid = 0
    },

    // 展示编辑商品分类名称的对话框
    async showEditCateDialog (id) {
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品名称失败！')
      }
      this.editCateForm = res.data
      this.editCateDialogVisible = true
    },
    // 监听对话框的关闭事件，重置表单数据
    editCateDialogClosed () {
      this.$refs.editCateFormRef.resetFields()
    },
    // 修改商品分类名称信息并提交
    editCateInfo () {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('categories/' + this.editCateForm.cat_id, { cat_name: this.editCateForm.cat_name })
        if (res.meta.status !== 200) {
          return this.$message.error('修改商品名称失败！')
        }
        this.$message.success('修改商品名称成功！')
        this.getCateList()
        this.editCateDialogVisible = false
      })
    },
    // 根据ID删除对应的商品
    async removeCateById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // console.log('确认删除')
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      this.$message.success('删除权限成功！')
      this.getCateList()
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
