<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary"
                     @click="addRolesDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolesList"
                border
                stripe>

        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row :class="['bdBottom',i1 === 0 ? 'bdTop' : '','vcenter']"
                    v-for="(item1,i1) in scope.row.children"
                    :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable
                        @close="removeRightByID(scope.row,item1.id)">{{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>

              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row :class="[i2 === 0 ? '' : 'bdTop','vcenter']"
                        v-for="(item2,i2) in item1.children"
                        :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success"
                            closable
                            @close="removeRightByID(scope.row,item2.id)">{{ item2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>

                  <!-- 通过for循环嵌套渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag type="danger"
                            closable
                            @close="removeRightByID(scope.row,item3.id)"
                            v-for="item3 in item2.children"
                            :key="item3.id">{{ item3.authName }}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>

            <!-- <pre>{{scope.row}}</pre> -->
          </template>
        </el-table-column>

        <!-- 索引列 -->
        <el-table-column type="index"
                         label="#"></el-table-column>
        <el-table-column label="角色名称"
                         prop="roleName"></el-table-column>
        <el-table-column label="角色描述"
                         prop="roleDesc"></el-table-column>
        <el-table-column label="操作"
                         width="300px">
          <template v-slot:default="scope">
            <el-button size="mini"
                       type="primary"
                       icon="el-icon-edit"
                       @click="showEditRolesDialog(scope.row.id)">编辑</el-button>
            <el-button size="mini"
                       type="danger"
                       icon="el-icon-delete"
                       @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button size="mini"
                       type="warning"
                       icon="el-icon-setting"
                       @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色的对话框 -->
    <el-dialog title="添加角色"
               :visible.sync="addRolesDialogVisible"
               width="50%"
               @close="addRolesDialogClosed()">
      <!-- 内容主体区域 -->
      <el-form :model="addRolesForm"
               :rules="addRolesFormRules"
               ref="addRolesFormRef"
               label-width="10·0px"
               status-icon>
        <el-form-item label="角色 ID"
                      prop="roleId">
          <el-input v-model="addRolesForm.roleId"></el-input>
        </el-form-item>
        <el-form-item label="角色名称"
                      prop="roleName">
          <el-input v-model="addRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="addRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addRolesDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addRole()">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改角色的对话框 -->
    <el-dialog title="修改角色"
               :visible.sync="editRolesDialogVisible"
               width="50%"
               @close="editRolesDialogClosed">
      <el-form :model="editRolesForm"
               :rules="editRolesFormRules"
               ref="editRolesFormRef"
               label-width="70px">
        <el-form-item label="角色 ID">
          <el-input v-model="editRolesForm.roleId"
                    prop="roleId"
                    disabled></el-input>
        </el-form-item>
        <el-form-item label="
                    角色名称"
                      prop="roleName">
          <el-input v-model="editRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editRolesInfo()">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限"
               :visible.sync="SetRightdialogVisible"
               width="50%"
               @close="setRightsDialogClosed()">
      <!-- 树形控件 -->
      <el-tree :data="rightsList"
               :props="RightsTreeProps"
               show-checkbox
               node-key="id"
               default-expand-all
               :default-checked-keys="defKeys"
               ref="treeRef"></el-tree>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="SetRightdialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="allotRights()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      rolesList: [],
      // 控制添加角色对话框的显示与隐藏
      addRolesDialogVisible: false,
      // 控制修改角色对话框的显示与隐藏
      editRolesDialogVisible: false,
      // 添加角色的表单数据
      addRolesForm: {
        roleId: '',
        roleName: '',
        roleDesc: ''
      },
      // 修改角色的表单数据
      editRolesForm: {
        roleId: '',
        roleName: '',
        roleDesc: ''
      },
      // 添加表单的验证规则对象
      addRolesFormRules: {
        roleId: [
          { required: true, message: '请输入角色ID', trigger: 'blur' },
          { min: 1, max: 10, message: '角色ID的长度在1-10个字符之间', trigger: 'blur' }
        ],
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 10, message: '角色名称的长度在2-10个字符之间', trigger: 'blur' }
        ]
      },
      // 添加表单的验证规则对象
      editRolesFormRules: {
        roleId: [
          { required: true, message: '请输入角色ID', trigger: 'blur' },
          { min: 1, max: 10, message: '角色ID的长度在1-10个字符之间', trigger: 'blur' }
        ],
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 10, message: '角色名称的长度在2-10个字符之间', trigger: 'blur' }
        ]
      },
      // 控制分配权限对话框的显示与隐藏
      SetRightdialogVisible: false,
      // 所有权限的数据
      rightsList: [],
      // 树形控件的属性绑定对象
      RightsTreeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点ID值数组
      defKeys: [],
      // 即将分配权限的角色Id
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色的列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
      // console.log(this.rolesList)
    },
    // 监听添加角色对话框的关闭事件
    addRolesDialogClosed () {
      this.$refs.addRolesFormRef.resetFields()
    },

    // 点击按钮，添加新角色
    addRole () {
      this.$refs.addRolesFormRef.validate(async valid => {
        if (!valid) return
        // 可以发起添加用户的网络请求
        const { data: res } = await this.$http.post('roles', this.addRolesForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败！')
        }
        this.$message.success('添加角色成功！')
        // 隐藏添加角色的对话框
        this.addRolesDialogVisible = false
        // 重新获取角色的列表数据
        this.getRolesList()
      })
    },

    // 展示编辑角色的对话框
    async showEditRolesDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色信息失败！')
      }
      this.editRolesForm = res.data
      this.editRolesDialogVisible = true
    },
    // 监听修改角色对话框的关闭事件
    editRolesDialogClosed () {
      this.$refs.editRolesFormRef.resetFields()
    },
    // 修改角色信息并提交
    editRolesInfo () {
      this.$refs.editRolesFormRef.validate(async valid => {
        if (!valid) return
        // 发起修改角色信息的数据请求
        const { data: res } = await this.$http.put('roles/' + this.editRolesForm.roleId, {
          roleName: this.editRolesForm.roleName,
          roleDesc: this.editRolesForm.roleDesc
        })

        if (res.meta.status !== 200) {
          return this.$message.error('更新角色信息失败！')
        }
        // 关闭对话框
        this.editRolesDialogVisible = false
        // 刷新数据列表
        this.getRolesList()
        // 提示修改成功
        this.$message.success('更新角色信息成功！')
      })
    },
    // 根据id删除对应的角色信息
    async removeRoleById (id) {
      // 弹框询问用户是否删除数据
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户点击确认删除，则返回值为字符串 confirm
      // 如果用户取消了删除，则返回值为字符串 cancel
      console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除用户失败！')
      this.$message.success('删除用户成功！')
      this.getRolesList()
    },
    // 根据ID删除对应的权限
    async removeRightByID (role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // console.log('确认删除')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      this.$message.error('已删除该权限')
      // this.getRolesList()
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败！')
      }
      // 把获取到的权限数据保存到 data 中
      this.rightsList = res.data
      // console.log(this.rightsList)

      // 递归获取三级节点的Id
      this.getLeafKeys(role, this.defKeys)

      this.SetRightdialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的ID，并保存到defKeys数组中
    getLeafKeys (node, arr) {
      // 如果当前node节点不包含children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightsDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      // console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }

      this.$message.success('分配权限成功')
      this.getRolesList()
      this.SetRightdialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdTop {
  border-top: 1px solid #eee;
}
.bdBottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
