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
    <el-row :gutter="12">
     <el-col :span="8">
         <el-input placeholder="请输入" v-model="queryInfo.query" clearable @clear='getGoodsList'>
             <el-button slot="append" icon='el-icon-search' @click="getGoodsList()" ></el-button>
         </el-input>
     </el-col>
     <el-col :span='4'>
         <el-button type='primary' @click="goAddpage">添加商品</el-button>
     </el-col>
    </el-row>

    <!-- table表格区域 -->
    <el-table :data='goodsList' border stripe>
        <el-table-column type='index' label="#"></el-table-column>
        <el-table-column label="商品名称" prop='goods_name'></el-table-column>
        <el-table-column label="商品价格" prop='goods_price' width='110px'></el-table-column>
        <el-table-column label="商品重量" prop='goods_weight' width='130px'></el-table-column>
        <el-table-column label="创建时间" prop='add_time' width='220px'>
            <template slot-scope="scope">
                {{scope.row.add_time*1000 | formatTimer}}
                <!-- {{dateFormat(scope.row.add_time ,'yyyy-MM-dd hh:mm:ss')}} -->
            </template>
        </el-table-column>
        <el-table-column label="操作" width='150px'>
            <template slot-scope="scope">
                <!-- <el-button type="primary" icon='el-icon-edit' size="mini" @click="changeById(scope.row.goods_id)">编辑</el-button> -->
                <el-button type="danger" icon='el-icon-delete' size="mini" @click="removeById(scope.row.goods_id)">删除</el-button>
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
    </div>
</template>
<script>
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      goodsList: [],
      total: 0
    }
  },
  created () {
    this.getGoodsList()
    console.log(this.goodsList.goods_name);
  },
   filters: {
        formatTimer: function(value) {
          let date = new Date(value);
          let y = date.getFullYear();
          let MM = date.getMonth() + 1;
          MM = MM < 10 ? "0" + MM : MM;
          let d = date.getDate();
          d = d < 10 ? "0" + d : d;
          let h = date.getHours();
          h = h < 10 ? "0" + h : h;
          let m = date.getMinutes();
          m = m < 10 ? "0" + m : m;
          let s = date.getSeconds();
          s = s < 10 ? "0" + s : s;
          return y + "-" + MM + "-" + d + " " + h + ":" + m;
        }
      },
  methods: {
    // 根据分页获取对应的商品列表
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败')
      }
      this.$message.success('获取商品列表成功！')
      this.goodsList = res.data.goods
      let addArray = [925,924,923,922,921,920,919,918,917,916,915,914,913,911,910,909,908,907,906]
     this.goodsList.forEach(element => {
       
       if(addArray.indexOf(element.goods_id) > -1) {
     element.add_time = 1634865257
       }
      
       
     });
      console.log(res);
      console.log(this.goodsList);
      this.total = res.data.total
      // console.log(res)
    },
    // 监听 pagesize 该变的事件
    handleSizeChange (newsize) {
      // console.log(newsize)
      this.queryInfo.pagesize = newsize
      this.getGoodsList()
    },

    // 监听 页面值 的改变
    handleCurrentChange (newpage) {
      // console.log(newpage)
      this.queryInfo.pagenum = newpage
      this.getGoodsList()
    },
    // 根据ID 编辑 属性
    async changeById(id) {
           var res = this.$http.put(`categories/${id}`,{})
    },

    // 根据ID 删除 属性
    async removeById (id) {
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

      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败！')
      }

      this.$message.success('删除用户成功')
      this.getGoodsList()
    },
    goAddpage () {
      this.$router.push('/goods/add')
    },
    dateFormat(date,format){
    var o = {
        "M+" : date.getMonth()+1, //month
        "d+" : date.getDate(),    //day
        "h+" : date.getHours(),   //hour
        "m+" : date.getMinutes(), //minute
        "s+" : date.getSeconds(), //second
        "q+" : Math.floor((date.getMonth()+3)/3),  //quarter
        "S" : date.getMilliseconds() //millisecond
    };
    if(/(y+)/.test(format)) format=format.replace(RegExp.$1, (date.getFullYear()+"").substr(4 - RegExp.$1.length));
    for(var k in o)
        if(new RegExp("("+ k +")").test(format))
            format = format.replace(RegExp.$1, RegExp.$1.length==1 ? o[k] :    ("00"+ o[k]).substr((""+ o[k]).length));
    return format;
}

// var now = new Date();
// var date = dateFormat(now,'yyyy-MM-dd hh:mm:ss')
  }
}
</script>
<style lang="less" scoped>
</style>
