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
       <el-row>
           <el-col>
               <el-button @click="getAddDialogVisible">添加</el-button>
           </el-col>
       </el-row>

       <!-- 表格 -->
       <tree-table class="tree-table" :data='catelist' :columns='columns' :selection-type='false'
       :expand-type='false' show-index index-text='#' border :show-row-hover='false'>

       <template slot="isOk" slot-scope='scope'>
           <i class="el-icon-circle-close" v-if="scope.row.cat_deleted === true" style="color: red"></i>
           <i class="el-icon-circle-check" v-else style="color: lightgreen"></i>
       </template>
       <!-- 排序模块 -->
       <template slot="order" slot-scope='scope'>
           <el-tag v-if="scope.row.cat_level === 0" >一级</el-tag>
           <el-tag v-else-if="scope.row.cat_level === 1" type='success'>二级</el-tag>
           <el-tag v-else type="warning">三级</el-tag>
       </template>
       <!-- 操作模块 -->
       <template slot="opt" slot-scope='scope'>
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="moveTreeTable(scope.row.cat_id)">编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="deletRolesById(scope.row.cat_id)"
              >删除</el-button
            >
       </template>
       </tree-table>

       <!-- 底部分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page.sync="querInfo.pagenum"
      :page-size="querInfo.pagesize"
      :page-sizes="[3, 6, 10, 15]"
      layout="total, prev, pager, next,sizes"
      :total="total">
    </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
  title="提示"
  :visible.sync="addDialogVisible"
  width="30%">
  <el-form :model="addCateForm" :rules="addCateRules" ref="addCateRuleRef" label-width="100px">
  <el-form-item label="分类名称" prop="cat_name">
    <el-input v-model="addCateForm.cat_name"></el-input>
  </el-form-item>
  <el-form-item label="父类名称">
    <el-cascader
    v-model="cascaderValue"
    :options="parentCateList"
    :props="cascaderProps"
    @change="cascaderChange" class="cascader"></el-cascader>
  </el-form-item>
  </el-form>
  <span slot="footer">
    <el-button @click="addDialogClosed">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
</el-dialog>
<!-- <div v-for=" item in parentCateList" :key="item.id">{{item}}</div> -->
<!-- 编辑按钮的对话框 -->
<el-dialog
  title="提示"
  :visible.sync="moveDialogVisible"
  width="30%">
  <el-form ref="moveCateRef" :model="moveCateform" label-width="80px">
  <el-form-item label="名称">
    <el-input v-model="moveCateform.cat_name"></el-input>
  </el-form-item>
  </el-form>

  <span slot="footer">
    <el-button @click="moveDialogClosed">取 消</el-button>
    <el-button type="primary" @click="moveCate(scope.row.cat_id)">确 定</el-button>

  </span>
  </el-dialog>
    </div>
</template>
<script>
export default {
  data () {
    return {
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      //   商品分类数据列表
      catelist: [],
      //   为table指定列的数据
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          prop: 'cat_deleted',
          //   将当前列定义成模板列
          type: 'template',
          template: 'isOk'
        },
        {
          label: '排序',
          prop: 'cat_level',
          //   将当前列定义成模板列
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          //   prop: 'cat_deleted',
          //   将当前列定义成模板列
          type: 'template',
          template: 'opt'
        }
      ],
      //   添加分类对话框的显示与隐藏
      addDialogVisible: false,
      //   表单的数据绑定对象
      addCateForm: {
        // 将要添加的分类的名称
        cat_name: '',
        //  父级分类的ID
        cat_pid: 0,
        //  分类的等级， 默认要添加的是一级分类
        cat_level: 0
      },
      // 表单的验证规则
      addCateRules: {},
      // 父级分类的列表
      parentCateList: [],
      // 指定联级选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      cascaderValue: [],
      // 编辑按钮的对话框 显示隐藏
      moveDialogVisible: false,
      // 编辑对话框的 数据源
      moveCateform: {
        cat_name: ''
      }
    }
  },
  created () {
    this.getCatelist()
  },
  methods: {
    //   获取商品分列列表
    async getCatelist () {
      const { data: res } = await this.$http.get('categories', { params: this.querInfo })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分列失败')
      }
      this.catelist = res.data.result
      this.total = res.data.total
      // console.log(this.catelist)
    },
    //
    handleSizeChange (newSize) {
      this.querInfo.pagesize = newSize
      this.getCatelist()
    },
    handleCurrentChange (newPage) {
      this.querInfo.pagenum = newPage
      this.getCatelist()
    },
    // 添加分类
    getAddDialogVisible () {
      this.addDialogVisible = true
      this.getParentCateList()
    },
    // 获取父类分类列表的请求
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父类分类列表的请求失败')
      }
      // console.log(res.data)
      this.parentCateList = res.data

      // console.log(this.parentCateList, '44444444444444444444')
    },
    // 选择框该变的函数
    cascaderChange () {
      // console.log(this.cascaderValue)
      if (this.cascaderValue.length > 0) {
        // 父级分类的ID
        this.addCateForm.cat_pid = this.cascaderValue[this.cascaderValue.length - 1]
        // 分类的等级
        this.addCateForm.cat_level = this.cascaderValue.length
      } else {
        // 父级分类的ID
        this.addCateForm.cat_pid = 0
        // 分类的等级
        this.addCateForm.cat_level = 0
      }
    },
    // 提交按钮  添加新的分类
    addCate () {
      this.cascaderChange()
      this.$refs.addCateRuleRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('获取新的分类失败')
        }
        this.$message.success('获取新的分类成功')
        this.getParentCateList()
      })
      this.addDialogVisible = false
      // console.log(this.addCateForm)
    },
    // 表单关闭按钮 ,重置表单数据
    addDialogClosed () {
      this.$refs.addCateRuleRef.resetFields()
      this.cascaderValue = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
      this.addDialogVisible = false
    },
    //  编辑按钮 事件
    moveTreeTable (id) {
      // console.log(id)
      this.moveDialogVisible = true
      this.moveCate(id)
    },
    // 编辑完成 提交按钮
    async moveCate (id) {
      const data = await this.$http.put(`categories/${id}`, { params: this.moveCateform })
      console.log(data)
    },
    // 删除分类 按钮
    async deletRolesById (id) {
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

      const { data: res } = await this.$http.delete('categories/' + id)
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败！')
      }

      this.$message.success('删除用户成功')
      this.getParentCateList()
    },
    moveDialogClosed() {

    }
  }

}
</script>
<style  scoped>
.tree-table {
    margin-top: 15px;
}

</style>
