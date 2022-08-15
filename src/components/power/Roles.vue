<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope='scope'>
            <!-- {{scope.row}} -->
            <el-row v-for="(item1, i1) in scope.row.children" :key='item1.id' :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']">
              <!-- 一级菜单  -->
              <el-col :span="6"  >
                <el-tag closable @close='removeRightById(scope.row, item1.id)'>{{item1.authName}}</el-tag>
              </el-col>
              <!-- 二级菜单 三级菜单 -->
              <el-col :span="18">
                <el-row v-for="(item2, i2) in item1.children" :key="item2.id" :class="['bdbottom', i2 === 0 ? '' : 'bdtop']">
                  <el-col :span='6'><el-tag closable type='success' @close='removeRightById(scope.row, item2.id)'>{{item2.authName}}</el-tag></el-col>
                  <el-col :span='18'><el-tag closable @close='removeRightById(scope.row, item3.id)' type='warning' v-for="(item3) in item2.children" :key="item3.id">{{item3.authName}}</el-tag></el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="deletRolesById(scope.row.id)"
              >删除</el-button
            >
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)"
              >分配</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>

   <!-- 展示分配权限的对话框 -->
  <el-dialog
  title="提示"
  :visible.sync="showRolesDialogVisible"
  width="30%"
  
  @close='setRightDialogClosed'>
  <!-- 树形控件 -->
  <el-tree :data="rightsList" :props="treeProps"  show-checkbox default-expand-all
  node-key='id' :default-checked-keys="defKeys" ref="treeRef"></el-tree>

  <span slot="footer" >
    <el-button @click="showRolesDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights" >确 定</el-button>
  </span>
</el-dialog>
<!-- 添加角色对话框 -->
      <el-dialog title="添加角色" :visible.sync="addDialogVisible" @close="addDislogClosed">
        <el-form :model="addRolesForm"  ref="addRolesForm" label-width="100px">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addRolesForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addRolesForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRolesUser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 编辑对话框 -->
    <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="30%">
      <el-form :model="editRolesForm" ref="editFormRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editFormInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      rolesList: [],
      showRolesDialogVisible: false,
      rightsList: [],
      addDialogVisible: false,
      // 编辑对话框的显示和隐藏
      editDialogVisible: false,
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      roleId: '',
      addRolesForm: {
          roleName: '',
          roleDesc: ''
        },
        editRolesForm: {},
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      // console.log(this.$http.get('roles'))
      // console.log(await this.$http.get('roles'))
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
      // console.log(this.rolesList[0].id)
    },
     // 添加角色
    addRolesUser() {
      this.$refs.addRolesForm.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRolesForm)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败!')
        }
        this.$message.success('添加角色成功!')
        this.getRolesList()
        this.addDialogVisible = false
      })
      // console.log(this.addRolesForm)
    },
    // 得到修改用户的信息
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色失败!')
      }
      this.editRolesForm = res.data
      // console.log(this.editRolesForm)
      this.editDialogVisible = true
    },
    // 清空添加角色对话框
    addDislogClosed() {
      this.$refs.addRolesForm.resetFields()
    },
    // 删除角色 操作
    async deletRolesById (id) {
      // console.log(id)
      // console.log(this.rolesList.id)
      const confirmResult = await this.$confirm(
        '此操作将永久删除该账号, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
          center: true
        }
      ).catch((err) => err)

      // 如果用户确定删除，返回值为字符串 confirm
      // 如果用户取消了删除，返回值为字符串 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }

      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败！')
      }

      this.$message.success('删除用户成功')
      this.getRolesList()
    },
    // 根据id删除对应的权限
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该账号, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
        center: true
      }).catch(err => err)

      // 如果用户确定删除，返回值为字符串 confirm
      // 如果用户取消了删除，返回值为字符串 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        this.$message.error('获取数据权限失败')
      }
      this.rightsList = res.data
      this.getLeafKeys(role, this.defKeys)
      this.showRolesDialogVisible = true
    },
    editFormInfo() {
      this.$refs.editFormRef.validate(valid => {
        if (!valid) return
        this.$http
          .put('roles/' + this.editRolesForm.roleId, {
            roleName: this.editRolesForm.roleName,
            roleDesc: this.editRolesForm.roleDesc
          })
          .then(res => {
            if (res.data.meta.status !== 200) {
              return this.$message.error('修改角色失败!')
            }
            this.$message.success('修改角色成功!')
            this.getRolesList()
          })
        this.editDialogVisible = false
      })
    },
    // 通过递归的形式，获取角色下的所有三级权限的id， 并保存到 defKeys数组中
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 关闭权限对话框 数组刷新
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async  allotRights () {
      const keys = [...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()]

      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }

      this.$message.success('分配权限成功')
      this.getRolesList()
      this.showRolesDialogVisible = false
    }
  }
}
</script>
<style scoped>
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
