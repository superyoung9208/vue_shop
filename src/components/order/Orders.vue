<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data="orders" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="order_number" label="订单编号"></el-table-column>
        <el-table-column prop="order_price" label="订单价格"></el-table-column>
        <el-table-column prop="order_pay" label="是否付款">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.order_pay !== '0'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货"></el-table-column>
        <el-table-column prop="create_time" label="下单时间">
          <template slot-scope="scope">{{scope.row.create_time | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作">
          <template>
            <el-button type="success" icon="el-icon-edit" @click="addressDialogVisible = true"></el-button>
            <el-button type="primary" icon="el-icon-location-outline" @click="showProgressDialog"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum - 0"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
      <!-- 修改地址对话框 -->
      <el-dialog title="修改地址" :visible.sync="addressDialogVisible" width="50%" @close="handleClose">
        <el-form
          :model="addressForm"
          :rules="addressFormRules"
          ref="addressFormRef"
          label-width="100px"
          label-position="left"
        >
          <el-form-item label="省市区/县" prop="address1">
            <el-cascader
              v-model="addressForm.address2"
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
          <el-button type="primary" @click="addressDialogVisible = false">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 查看物流信息对话框 -->
      <el-dialog title="物流信息" :visible.sync="progressDialogVisible" width="50%">
        <el-timeline :reverse="false">
          <el-timeline-item
            v-for="(activity, index) in progressData"
            :key="index"
            :timestamp="activity.time"
          >{{activity.context}}</el-timeline-item>
        </el-timeline>
        <span slot="footer" class="dialog-footer">
          <el-button @click="progressDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="progressDialogVisible = false">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import cityData from './citydata.js'
export default {
  data() {
    return {
      orders: [],
      progressData: [],
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      addressDialogVisible: false,
      // 物流对话框
      progressDialogVisible: false,
      cityData,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请输入省市区/县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getOrederList()
  },
  methods: {
    // 获取订单列表数据
    async getOrederList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$Message.error('获取订单列表数据失败！')
      }
      this.$Message.success('获取订单数据成功！')
      this.orders = res.data.goods
      this.total = res.data.total
      this.queryInfo.pagenum = res.data.pagenum
    },
    // 每页获取条目个数
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getOrederList()
    },
    // 改变分页
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getOrederList()
    },
    // 关闭对话框
    handleClose() {
      this.$refs.addressFormRef.resetFields()
    },
    // 显示物流信息对话框
    async showProgressDialog() {
      const { data: res } = await this.$http.get('kuaidi/804909574412544580')
      if (res.meta.status !== 200) {
        return this.$Message.error('获取物流信息失败')
      }
      this.progressData = res.data
      this.progressDialogVisible = true
      console.log(res.data)
    }
  }
}
</script>

<style lang="less" scoped>
</style>
