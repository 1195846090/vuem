<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col :span="6">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>

      <!-- 订单列表 -->
      <el-table :data="orderList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单押金" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" size="mini" v-if="scope.row.pay_status == '1'">已付款</el-tag>
            <el-tag type="danger" size="mini" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"></el-table-column>
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">
            {{+('162'+scope.row.create_time) | dataFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot slot-scope="scope">
            <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog"></el-button>
            <el-button
              type="success"
              size="mini"
              icon="el-icon-location"
              @click="showProgressDialog"
              v-if="scope.row.is_send == '是'"
            ></el-button>
            <el-button
              type="info"
              size="mini"
              icon="el-icon-location"
              v-else
              @click="messageinfo"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 编辑对话框 -->
    <el-dialog
      title="修改地址"
      :visible.sync="addressDialogVisible"
      width="50%"
      @close="addressDialogClosed"
    >
      <el-form
        :model="addressForm"
        :rules="addressFormRules"
        ref="addressFormRef"
        label-width="100px"
      >
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader
            v-model="addressForm.address1"
            :options="cityData"
            :props="{ expandTrigger: 'hover' }"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="closereids">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 展示物流进度对话框 -->
    <el-dialog title="查看物流进度" :visible.sync="progressDialogVisible" width="50%">
    <!-- 时间线 -->
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in progressInfo"
          :key="index"
          :timestamp="activity.time"
        >{{activity.context}}</el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'

export default {
  data () {
    return {
      // 订单列表查询参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      // 订单列表
      orderList: [],
      // 修改地址对话框
      addressDialogVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      // 物流进度对话框
      progressDialogVisible: false,
      // 物流进度
      progressInfo: [],
    progress1: [
      {
        time: "2021-05-23 09:39:00",
        ftime: "2021-05-23 09:39:00",
        context: "已签收,感谢使用顺丰,期待再次为您服务",
        location: ""
      },
      {
        time: "2021-05-23 08:23:00",
        ftime: "2021-05-23 08:23:00",
        context: "[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件",
        location: ""
      },
      {
        time: "2021-05-23 07:32:00",
        ftime: "2021-05-23 07:32:00",
        context: "快件到达 [北京海淀育新小区营业点]",
        location: ""
      },
      {
        time: "2021-05-23 02:03:00",
        ftime: "2021-05-23 02:03:00",
        context: "快件在[北京顺义集散中心]已装车,准备发往 [北京海淀育新小区营业点]",
        location: ""
      },
      {
        time: "2021-05-22 23:05:00",
        ftime: "2021-05-22 23:05:00",
        context: "快件到达 [北京顺义集散中心]",
        location: ""
      },
      {
        time: "2021-05-22 21:21:00",
        ftime: "2021-05-22 21:21:00",
        context: "快件在[北京宝胜营业点]已装车,准备发往 [北京顺义集散中心]",
        location: ""
      },
      {
        time: "2021-05-22 13:07:00",
        ftime: "2021-05-22 13:07:00",
        context: "顺丰速运 已收取快件",
        location: ""
      },
      {
        time: "2021-05-22 12:25:03",
        ftime: "2021-05-22 12:25:03",
        context: "卖家发货",
        location: ""
      },
      {
        time: "2021-05-22 12:22:24",
        ftime: "2021-05-22 12:22:24",
        context: "您的订单将由HLA（北京海淀区清河中街店）门店安排发货。",
        location: ""
      },
      {
        time: "2021-05-21 21:36:04",
        ftime: "2021-05-21 21:36:04",
        context: "商品已下单",
        location: ""
      }
    ],
    progress2: [
      {
        time: "2021-05-23 09:39:00",
        ftime: "2021-05-23 09:39:00",
        context: "已签收,感谢使用顺丰,期待再次为您服务",
        location: ""
      },
      {
        time: "2021-05-22 12:25:03",
        ftime: "2021-05-22 12:25:03",
        context: "卖家发货",
        location: ""
      },
      {
        time: "2021-05-22 12:22:24",
        ftime: "2021-05-22 12:22:24",
        context: "您的订单将由HLA（湖南长沙星沙区卜蜂莲花店）门店安排发货。",
        location: ""
      },
      {
        time: "2021-05-21 21:36:04",
        ftime: "2021-05-21 21:36:04",
        context: "商品已下单",
        location: ""
      }
    ],
    progress3: [
       {
        time: "2021-05-22 15:30:44",
        ftime: "2021-05-22 15:30:44",
        context: "货物已揽收",
        location: ""
      },
      {
        time: "2021-05-21 21:36:04",
        ftime: "2021-05-21 21:36:04",
        context: "商品已下单",
        location: ""
      }
    ],
    progress4: [
      {
        time: "2021-05-23 07:32:00",
        ftime: "2021-05-23 07:32:00",
        context: "快件到达 [井冈分区望安小区营业点]",
        location: ""
      },
      {
        time: "2021-05-23 02:03:00",
        ftime: "2021-05-23 02:03:00",
        context: "快件在[长沙顺义集散中心]已装车,准备发往 [井冈分区望安小区营业点]",
        location: ""
      },
      {
        time: "2021-05-22 23:05:00",
        ftime: "2021-05-22 23:05:00",
        context: "快件到达 [长沙顺义集散中心]",
        location: ""
      },
      {
        time: "2021-05-22 21:21:00",
        ftime: "2021-05-22 21:21:00",
        context: "快件在[长沙宝胜营业点]已装车,准备发往 [长沙顺义集散中心]",
        location: ""
      },
      {
        time: "2021-05-22 13:07:00",
        ftime: "2021-05-22 13:07:00",
        context: "顺丰速运 已收取快件",
        location: ""
      },
      {
        time: "2021-05-22 12:25:03",
        ftime: "2021-05-22 12:25:03",
        context: "卖家发货",
        location: ""
      },
      {
        time: "2021-05-22 12:22:24",
        ftime: "2021-05-22 12:22:24",
        context: "您的订单将由HLA（北京海淀区清河中街店）门店安排发货。",
        location: ""
      },
      {
        time: "2021-05-21 21:36:04",
        ftime: "2021-05-21 21:36:04",
        context: "商品已下单",
        location: ""
      }
    ],
    progress5: [
      {
        time: "2021-05-22 23:05:00",
        ftime: "2021-05-22 23:05:00",
        context: "快件到达 [长沙顺义集散中心] 快件预计5-10号到达",
        location: ""
      },
      {
        time: "2021-05-22 13:07:00",
        ftime: "2021-05-22 13:07:00",
        context: "顺丰速运 已收取快件",
        location: ""
      },
      {
        time: "2021-05-22 12:25:03",
        ftime: "2021-05-22 12:25:03",
        context: "卖家发货",
        location: ""
      },
      {
        time: "2021-05-22 12:22:24",
        ftime: "2021-05-22 12:22:24",
        context: "您的订单将由HLA（深圳龙华区金茂店）门店安排发货。",
        location: ""
      },
      {
        time: "2021-05-21 21:36:04",
        ftime: "2021-05-21 21:36:04",
        context: "商品已下单",
        location: ""
      }
    ],
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
        return this.$message.error('获取订单列表失败！')
      }
      this.total = res.data.total
      this.orderList = res.data.goods
      console.log(this.orderList);
    },
    // 分页
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange (newSize) {
      this.queryInfo.pagenum = newSize
      this.getOrderList()
    },
    showEditDialog () {
      this.addressDialogVisible = true
    },
    addressDialogClosed () {
      this.$refs.addressFormRef.resetFields()
    },
    showProgressDialog () {
      let value = Math.floor((Math.random()*10)+1)
      if(value==1 || value ==2){
        this.progressInfo = this.progress1
      }else if(value==3 || value ==4){
        this.progressInfo = this.progress2
      }else if(value==5 || value ==6){
        this.progressInfo = this.progress3
      }else if(value==7 || value ==8){
        this.progressInfo = this.progress4
      }else{
        this.progressInfo = this.progress5
      }
      const lth = this.progressInfo.length
      this.progressInfo[lth-1].time ="2021-05-20 20:08:53"
       this.$message.success('获取物流进度成功!')
      this.progressDialogVisible = true
    },
    closereids() {
      if(this.addressForm.address1 && this.addressForm.address2){
        this.addressDialogVisible = false;
        this.$message.success('修改地址成功')
      }else{
        this.$message.info('请填写完整地址')
      }
    },
    messageinfo() {
      this.$message.error('订单暂无物流信息')
    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
@import '../../plugins/timeline/timeline.css';
@import '../../plugins/timeline-item/timeline-item.css';
</style>
