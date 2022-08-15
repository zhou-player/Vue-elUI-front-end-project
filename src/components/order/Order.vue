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
      <!-- <el-row>
        <el-col :span='8'>
          <el-input
            placeholder="请输入内容"
          >
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row> -->

      <!-- 订单列表数据 -->
      <el-table :data="orderlist" border stripe>
          <el-table-column type='index'></el-table-column>
          <el-table-column label="订单编号" prop='order_number'></el-table-column>
          <el-table-column label="订单价格" prop='order_price'></el-table-column>
          <el-table-column label="是否付款" prop='pay_status'>
              <template slot-scope="scope">
                  <el-tag type='success' v-if="scope.row.pay_status === 1">已付款</el-tag>
                  <el-tag type='danger' v-else>未付款</el-tag>

              </template>
          </el-table-column>
          <el-table-column label="是否发货" prop='is_send'></el-table-column>
          <el-table-column label="下单时间" prop='create_time'>
              <template slot-scope="scope">
                  {{scope.row.create_time*1000 | dateFormat }}
              </template>
          </el-table-column>
          <el-table-column label="操作">
              <template>
              <el-button size="small" type="primary" icon="el-icon-edit" @click="showBox"></el-button>
              <el-button size="small" type="success" icon="el-icon-location" @click="showProgressBox"></el-button>
              </template>
          </el-table-column>

      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="100"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 对话框 -->
    <el-dialog
  title="修改地址"
  :visible.sync="addressDialogVisible"
  width="30%"
  @close='addressDialogClosed'>
  <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
  <el-form-item label="省市区县" prop="address1">
      <el-cascader  :options="options" v-model="addressForm.address1" @change="chooseArea" class="el-cascader"></el-cascader>
  </el-form-item>
  <el-form-item label="详细地址" prop="address2">
    <el-input v-model="addressForm.address2"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addressDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addressDialogVisible = false">确 定</el-button>
  </span>
</el-dialog>

<!-- 快递对话框 -->
<el-dialog
  title="这是一段物流信息"
  :visible.sync="ProgressDialogVisible"
  width="30%">
  <el-table :data="kuaidilist" border stripe>
          <el-table-column type='index'></el-table-column>
          <el-table-column label="2018-05-10 09:39:00" ></el-table-column>
          <el-table-column label="[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件" ></el-table-column>
          <el-table-column label="已签收,感谢使用顺丰,期待再次为您服务" ></el-table-column>
          

  </el-table>
  <span slot="footer" class="dialog-footer">
    <el-button @click="ProgressDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="ProgressDialogVisible = false">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>
<script>
// 省市区
import { regionData } from 'element-china-area-data'
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        // 当前的页数
        pagenum: 1,
        // 当前每页显示多少条数据
        pagesize: 10
      },
      //
      orderlist: [],
      //
      kuaidilist: { data: 
      {
        "time": "2018-05-10 09:39:00",
        "ftime": "2018-05-10 09:39:00",
        "context": "已签收,感谢使用顺丰,期待再次为您服务",
        "location": ""
      }},
      //
      total: 0,
      //   对话框
      addressDialogVisible: false,
      ProgressDialogVisible: false,
      options: regionData,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [{ required: true, message: '请选择省市区县', trigger: 'blur' }],
        address2: [{ required: true, message: '请输入详细地址', trigger: 'blur' }]
      }

    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败')
      }

      this.total = res.data.total
      this.orderlist = res.data.goods
      console.log(res)
    },
    // 监听 pagesize 该变的事件
    handleSizeChange (newsize) {
      console.log(newsize)
      this.queryInfo.pagesize = newsize
      this.getOrderList()
    },

    // 监听 页面值 的改变
    handleCurrentChange (newpage) {
      console.log(newpage)
      this.queryInfo.pagenum = newpage
      this.getOrderList()
    },
    // 展示修改对话框
    showBox () {
      this.addressDialogVisible = true
    },
    //
    addressDialogClosed () {
      this.$refs.addressFormRef.resetFields()
    },
    async showProgressBox () {
      // const { data: res } = await this.$http.get('/kuaidi/804909574412544580')
      // console.log(res)
      // if (res.meta.status !== 200) {
      //   return this.$message.error('未获取到物流信息')
      // }
      this.ProgressDialogVisible = true
      console.log('sss')
    },
    chooseArea(text) {
          console.log(text);
    }
  }
}
</script>
<style scoped>
.el-cascader-panel{
  height: 210px;
}
</style>
