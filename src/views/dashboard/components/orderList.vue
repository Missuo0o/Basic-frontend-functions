<template>
  <div>
    <div class="container homecon">
      <h2 class="homeTitle homeTitleBtn">
        Order Information
        <ul class="conTab">
          <li
            v-for="(item, index) in tabList"
            :key="index"
            :class="activeIndex === index ? 'active' : ''"
            @click="handleClass(index)"
          >
            <el-badge
              :class="item.num >= 10 ? 'badgeW' : ''"
              :hidden="!([2, 3].includes(item.value) && item.num)"
              :value="item.num > 99 ? '99+' : item.num"
              class="item"
            >{{ item.label }}
            </el-badge
            >
          </li>
        </ul>
      </h2>
      <div class="">
        <div v-if="orderData.length > 0">
          <el-table
            :data="orderData"
            class="tableBox"
            stripe
            style="width: 100%"
            @row-click="handleTable"
          >
            <el-table-column label="Order Number" prop="number"></el-table-column>
            <el-table-column label="Dishes">
              <template slot-scope="scope">
                <div class="ellipsisHidden">
                  <el-popover
                    :content="scope.row.orderDishes"
                    placement="top-start"
                    title=""
                    trigger="hover"
                    width="200"
                  >
                    <span slot="reference">{{ scope.row.orderDishes }}</span>
                  </el-popover>
                </div>
              </template>
            </el-table-column>
            <el-table-column
              :class-name="dialogOrderStatus === 2 ? 'address' : ''"
              label="Address"
            >
              <template slot-scope="scope">
                <div class="ellipsisHidden">
                  <el-popover
                    :content="scope.row.address"
                    placement="top-start"
                    title=""
                    trigger="hover"
                    width="200"
                  >
                    <span slot="reference">{{ scope.row.address }}</span>
                  </el-popover>
                </div>
              </template>
            </el-table-column>

            <el-table-column
              class-name="orderTime"
              label="ETA"
              min-width="130"
              prop="estimatedDeliveryTime"
              sortable
            >
            </el-table-column>
            <el-table-column label="Amount" prop="amount"></el-table-column>
            <el-table-column label="Remark">
              <template slot-scope="scope">
                <div class="ellipsisHidden">
                  <el-popover
                    :content="scope.row.remark"
                    placement="top-start"
                    title=""
                    trigger="hover"
                    width="200"
                  >
                    <span slot="reference">{{ scope.row.remark }}</span>
                  </el-popover>
                </div>
              </template>
            </el-table-column>
            <el-table-column
              v-if="status === 3"
              align="center"
              label="Tableware"
              min-width="80"
              prop="tablewareNumber"
            >
            </el-table-column>
            <el-table-column
              :class-name="dialogOrderStatus === 0 ? 'operate' : 'otherOperate'"
              :min-width="
                [2, 3].includes(dialogOrderStatus)
                  ? 130
                  : [0].includes(dialogOrderStatus)
                  ? 140
                  : 'auto'
              "
              align="center"
              label="Operate"
            >
              <template slot-scope="{ row }">
                <!-- <el-divider direction="vertical" /> -->
                <div class="before">
                  <el-button
                    v-if="row.status === 2"
                    class="blueBug"
                    type="text"
                    @click="
                      orderAccept(row, $event), (isTableOperateBtn = true)
                    "
                  >
                    Accept
                  </el-button>
                  <el-button
                    v-if="row.status === 3"
                    class="blueBug"
                    type="text"
                    @click="cancelOrDeliveryOrComplete(3, row.id, $event)"
                  >
                    Delivery
                  </el-button>
                </div>
                <div class="middle">
                  <el-button
                    v-if="row.status === 2"
                    class="delBut"
                    type="text"
                    @click="
                      orderReject(row, $event), (isTableOperateBtn = true)
                    "
                  >
                    Reject
                  </el-button>
                  <el-button
                    v-if="[1, 3, 4, 5].includes(row.status)"
                    class="delBut"
                    type="text"
                    @click="cancelOrder(row, $event)"
                  >
                    Cancel
                  </el-button>
                </div>
                <div class="after">
                  <el-button
                    class="blueBug non"
                    type="text"
                    @click="goDetail(row.id, row.status, row, $event)"
                  >
                    View
                  </el-button>
                </div>
              </template>
            </el-table-column>
          </el-table>
        </div>
        <Empty v-else :is-search="isSearch" />
        <el-pagination
          v-if="counts > 10"
          :page-size="pageSize"
          :page-sizes="[10, 20, 30, 40]"
          :total="counts"
          class="pageList"
          layout="total, sizes, prev, pager, next, jumper"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </div>
    <!-- 查看弹框部分 -->
    <el-dialog
      :before-close="handleClose"
      :visible.sync="dialogVisible"
      class="order-dialog"
      title="Order Information"
      width="53%"
    >
      <el-scrollbar style="height: 100%">
        <div class="order-top">
          <div>
            <div style="display: inline-block">
              <label style="font-size: 16px">Order Number</label>
              <div class="order-num">
                {{ diaForm.number }}
              </div>
            </div>
            <div
              :class="{ status3: [3, 4].includes(dialogOrderStatus) }"
              class="order-status"
              style="display: inline-block"
            >
              {{
                orderList.filter((item) => item.value === dialogOrderStatus)[0]
                  .label
              }}
            </div>
          </div>
          <p><label>Order Time:</label>{{ diaForm.orderTime }}</p>
        </div>

        <div class="order-middle">
          <div class="user-info">
            <div class="user-info-box">
              <div class="user-name">
                <label>Username:</label>
                <span>{{ diaForm.consignee }}</span>
              </div>
              <div class="user-phone">
                <label>Phone:</label>
                <span>{{ diaForm.phone }}</span>
              </div>
              <div
                v-if="[2, 3, 4, 5].includes(dialogOrderStatus)"
                class="user-getTime"
              >
                <label>{{
                    dialogOrderStatus === 5 ? 'Delivery time：' : 'ETA：'
                  }}</label>
                <span>{{
                    dialogOrderStatus === 5
                      ? diaForm.deliveryTime
                      : diaForm.estimatedDeliveryTime
                  }}</span>
              </div>
              <div class="user-address">
                <label>Address:</label>
                <span>{{ diaForm.address }}</span>
              </div>
            </div>
            <div
              :class="{ orderCancel: dialogOrderStatus === 6 }"
              class="user-remark"
            >
              <div>{{ dialogOrderStatus === 6 ? 'Reason' : 'Remark' }}</div>
              <span>{{
                  dialogOrderStatus === 6
                    ? diaForm.cancelReason || diaForm.rejectionReason
                    : diaForm.remark
                }}</span>
            </div>
          </div>

          <div class="dish-info">
            <div class="dish-label">Dish</div>
            <div class="dish-list">
              <div
                v-for="(item, index) in diaForm.orderDetailList"
                :key="index"
                class="dish-item"
              >
                <span class="dish-name">{{ item.name }}</span>
                <span class="dish-num">x{{ item.number }}</span>
                <span class="dish-price"
                >￥{{ item.amount ? item.amount.toFixed(2) : '' }}</span
                >
              </div>
            </div>
            <div class="dish-all-amount">
              <label>Dish Subtotal</label>
              <span
              >￥{{
                  (diaForm.amount - 6 - diaForm.packAmount).toFixed(2)
                }}</span
              >
            </div>
          </div>
        </div>

        <div class="order-bottom">
          <div class="amount-info">
            <div class="amount-label">Fee</div>
            <div class="amount-list">
              <div class="dish-amount">
                <span class="amount-name">Dish Subtotal:</span>
                <span class="amount-price"
                >￥{{
                    ((diaForm.amount - 6 - diaForm.packAmount).toFixed(2) *
                      100) /
                    100
                  }}</span
                >
              </div>
              <div class="send-amount">
                <span class="amount-name">Delivery Fee:</span>
                <span class="amount-price">￥{{ 6 }}</span>
              </div>
              <div class="package-amount">
                <span class="amount-name">Packing Fee:</span>
                <span class="amount-price"
                >￥{{
                    diaForm.packAmount
                      ? (diaForm.packAmount.toFixed(2) * 100) / 100
                      : ''
                  }}</span
                >
              </div>
              <div class="all-amount">
                <span class="amount-name">Total:</span>
                <span class="amount-price"
                >￥{{
                    diaForm.amount
                      ? (diaForm.amount.toFixed(2) * 100) / 100
                      : ''
                  }}</span
                >
              </div>
              <div class="pay-type">
                <span class="pay-name">Payment:</span>
                <span class="pay-value">{{
                    diaForm.payMethod === 1 ? 'Wechat Pay' : 'Alipay'
                  }}</span>
              </div>
              <div class="pay-time">
                <span class="pay-name">Payment Time:</span>
                <span class="pay-value">{{ diaForm.checkoutTime }}</span>
              </div>
            </div>
          </div>
        </div>
      </el-scrollbar>
      <span v-if="dialogOrderStatus !== 6" slot="footer" class="dialog-footer">
        <el-checkbox
          v-if="dialogOrderStatus === 2 && status === 2"
          v-model="isAutoNext"
        >Next one</el-checkbox
        >
        <el-button
          v-if="dialogOrderStatus === 2"
          @click="orderReject(row, $event), (isTableOperateBtn = false)"
        >Reject</el-button
        >
        <el-button
          v-if="dialogOrderStatus === 2"
          type="primary"
          @click="orderAccept(row, $event), (isTableOperateBtn = false)"
        >Accept</el-button
        >

        <el-button
          v-if="[1, 3, 4, 5].includes(dialogOrderStatus)"
          @click="dialogVisible = false"
        >Back</el-button
        >
        <el-button
          v-if="dialogOrderStatus === 3"
          type="primary"
          @click="cancelOrDeliveryOrComplete(3, row.id, $event)"
        >Delivery</el-button
        >
        <el-button
          v-if="dialogOrderStatus === 4"
          type="primary"
          @click="cancelOrDeliveryOrComplete(4, row.id, $event)"
        >Done</el-button
        >
        <el-button
          v-if="[1].includes(dialogOrderStatus)"
          type="primary"
          @click="cancelOrder(row, $event)"
        >Cancel</el-button
        >
      </span>
    </el-dialog>
    <!-- end -->
    <!-- 拒单，取消弹窗 -->
    <el-dialog
      :before-close="() => ((cancelDialogVisible = false), (cancelReason = ''))"
      :title="cancelDialogTitle"
      :visible.sync="cancelDialogVisible"
      class="cancelDialog"
      width="42%"
    >
      <el-form label-width="90px">
        <el-form-item :label="cancelDialogTitle">
          <el-select
            v-model="cancelReason"
          >
            <el-option
              v-for="(item, index) in cancelDialogTitle === 'Cancel'
                ? cancelrReasonList
                : cancelOrderReasonList"
              :key="index"
              :label="item.label"
              :value="item.label"
            />
          </el-select>
        </el-form-item>
        <el-form-item v-if="cancelReason === 'Custom'" label="Reason:">
          <el-input
            v-model.trim="remark"
            maxlength="50"
            type="textarea"
          />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click=";(cancelDialogVisible = false), (cancelReason = '')"
        >Cancel</el-button
        >
        <el-button type="primary" @click="confirmCancel">Submit</el-button>
      </span>
    </el-dialog>
    <!-- end -->
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'
import Empty from '@/components/Empty/index.vue'
import {
  completeOrder,
  deliveryOrder,
  getOrderDetailPage,
  orderAccept,
  orderCancel,
  orderReject,
  queryOrderDetailById
} from '@/api/order'

@Component({
  name: 'Orderview',
  components: {
    Empty
  }
})
export default class extends Vue {
  @Prop({ default: '' }) orderStatics!: any

  private orderId = '' //订单号
  private dialogOrderStatus = 0 //弹窗所需订单状态，用于详情展示字段
  private activeIndex = 0

  private dialogVisible = false //详情弹窗
  private cancelDialogVisible = false //取消，拒单弹窗
  private cancelDialogTitle = '' //取消，拒绝弹窗标题
  private cancelReason = ''
  private remark = '' //自定义原因
  private diaForm = []
  private row = {}
  private isAutoNext = true
  private isSearch: boolean = false
  private counts = 0
  private page: number = 1
  private pageSize: number = 10
  private status = 2
  private orderData = []
  private isTableOperateBtn = true
  private cancelOrderReasonList = [
    {
      value: 1,
      label: 'Higher order volume'
    },
    {
      value: 2,
      label: 'The dish is sold out'
    },
    {
      value: 3,
      label: 'The restaurant is closed'
    },
    {
      value: 0,
      label: 'Custom'
    }
  ]

  private cancelrReasonList = [
    {
      value: 1,
      label: 'Higher order volume'
    },
    {
      value: 2,
      label: 'The dish is sold out'
    },
    {
      value: 3,
      label: 'Insufficient riders to deliver'
    },
    {
      value: 4,
      label: 'Customer call cancellation'
    },
    {
      value: 0,
      label: 'Custom'
    }
  ]
  private orderList = [
    {
      label: 'All Orders',
      value: 0
    },
    {
      label: 'Pending Payment',
      value: 1
    },
    {
      label: 'Pending Order',
      value: 2
    },
    {
      label: 'Awaiting delivery',
      value: 3
    },
    {
      label: 'Delivering',
      value: 4
    },
    {
      label: 'Completed',
      value: 5
    },
    {
      label: 'Cancelled',
      value: 6
    }
  ]

  get tabList() {
    return [
      {
        label: 'Pending Order',
        value: 2,
        num: this.orderStatics.toBeConfirmed
      },
      {
        label: 'Awaiting delivery',
        value: 3,
        num: this.orderStatics.confirmed
      }
    ]
  }

  created() {
    this.getOrderListData(this.status)
  }

  // // 获取订单数据
  async getOrderListData(status) {
    const params = {
      page: this.page,
      pageSize: this.pageSize,
      status: status
    }
    const data = await getOrderDetailPage(params)
    this.orderData = data.data.data.records
    this.counts = data.data.data.total
    this.$emit('getOrderListBy3Status')
    if (
      this.dialogOrderStatus === 2 &&
      this.status === 2 &&
      this.isAutoNext &&
      !this.isTableOperateBtn &&
      data.data.records.length > 1
    ) {
      const row = data.data.records[0]
      this.goDetail(row.id, row.status, row, row)
    } else {
      return null
    }
  }

  //接单
  orderAccept(row: any, event) {
    event.stopPropagation()
    this.orderId = row.id
    this.dialogOrderStatus = row.status
    orderAccept({ id: this.orderId })
      .then((res) => {
        if (res.data.code === 1) {
          this.$message.success('Success!')
          this.orderId = ''
          // this.dialogOrderStatus = 0
          this.dialogVisible = false
          this.getOrderListData(this.status)
        } else {
          this.$message.error(res.data.msg)
        }
      })
      .catch((err) => {
        this.$message.error('error：' + err.message)
      })
  }

  //打开取消订单弹窗
  cancelOrder(row: any, event) {
    event.stopPropagation()
    this.cancelDialogVisible = true
    this.orderId = row.id
    this.dialogOrderStatus = row.status
    this.cancelDialogTitle = 'Cancel'
    this.dialogVisible = false
    this.cancelReason = ''
  }

  //打开拒单弹窗
  orderReject(row: any, event) {
    event.stopPropagation()
    this.cancelDialogVisible = true
    this.orderId = row.id
    this.dialogOrderStatus = row.status
    this.cancelDialogTitle = 'Reject'
    this.dialogVisible = false
    this.cancelReason = ''
  }

  //确认取消或拒绝订单并填写原因
  confirmCancel(type) {
    if (!this.cancelReason) {
      return this.$message.error(`${this.cancelDialogTitle}`)
    } else if (this.cancelReason === 'Custom' && !this.remark) {
      return this.$message.error(`${this.cancelDialogTitle}`)
    }

    ;(this.cancelDialogTitle === 'Cancel' ? orderCancel : orderReject)({
      id: this.orderId,
      // eslint-disable-next-line standard/computed-property-even-spacing
      [this.cancelDialogTitle === 'Cancel' ? 'cancelReason' : 'rejectionReason']:
        this.cancelReason === 'Custom' ? this.remark : this.cancelReason
    })
      .then((res) => {
        if (res.data.code === 1) {
          this.$message.success('Success!')
          this.cancelDialogVisible = false
          this.orderId = ''
          // this.dialogOrderStatus = 0
          this.getOrderListData(this.status)
        } else {
          this.$message.error(res.data.msg)
        }
      })
      .catch((err) => {
        this.$message.error('error：' + err.message)
      })
  }

  // 派送，完成
  cancelOrDeliveryOrComplete(status: number, id: string, event) {
    event.stopPropagation()
    const params = {
        status,
        id
      }
    ;(status === 3 ? deliveryOrder : completeOrder)(params)
      .then((res) => {
        if (res.data.code === 1) {
          this.$message.success('Success!')
          this.orderId = ''
          // this.dialogOrderStatus = 0
          this.dialogVisible = false
          this.getOrderListData(this.status)
        } else {
          this.$message.error(res.data.msg)
        }
      })
      .catch((err) => {
        this.$message.error('error：' + err.message)
      })
  }

  // 查看详情
  async goDetail(id: any, status: number, row: any, event) {
    event.stopPropagation()
    // console.log(111, index, row)
    this.diaForm = []
    this.dialogVisible = true
    this.dialogOrderStatus = status
    const { data } = await queryOrderDetailById({ orderId: id })
    this.diaForm = data.data
    this.row = row
  }

  // 关闭弹层
  handleClose() {
    this.dialogVisible = false
  }

  // tab切换
  handleClass(index) {
    this.activeIndex = index
    if (index === 0) {
      this.status = 2
      this.getOrderListData(2)
    } else {
      this.status = 3
      this.getOrderListData(3)
    }
  }

  // 触发table某一行
  handleTable(row, column, event) {
    event.stopPropagation()
    this.goDetail(row.id, row.status, row, event)
  }

  // 分页
  private handleSizeChange(val: any) {
    this.pageSize = val
    this.getOrderListData(this.status)
  }

  private handleCurrentChange(val: any) {
    this.page = val
    this.getOrderListData(this.status)
  }
}
</script>
<style lang="scss" scoped>
.dashboard-container.home .homecon {
  margin-bottom: 0;
}

.order-top {
  // height: 80px;
  border-bottom: 1px solid #e7e6e6;
  padding-bottom: 26px;
  padding-left: 22px;
  padding-right: 22px;
  // margin: 0 30px;
  display: flex;
  justify-content: space-between;
  align-items: center;

  .order-status {
    width: 57.25px;
    height: 27px;
    background: #333333;
    border-radius: 13.5px;
    color: white;
    margin-left: 19px;
    text-align: center;
    line-height: 27px;
  }

  .status3 {
    background: #f56c6c;
  }

  p {
    color: #333;

    label {
      color: #666;
    }
  }

  .order-num {
    font-size: 16px;
    color: #2a2929;
    font-weight: bold;
    display: inline-block;
  }
}

.order-middle {
  .user-info {
    min-height: 140px;
    background: #fbfbfa;
    margin-top: 23px;

    padding: 20px 43px;
    color: #333;

    .user-info-box {
      min-height: 55px;
      display: flex;
      flex-wrap: wrap;

      .user-name {
        flex: 67%;
      }

      .user-phone {
        flex: 33%;
      }

      .user-getTime {
        margin-top: 14px;
        flex: 80%;

        label {
          margin-right: 3px;
        }
      }

      label {
        margin-right: 17px;
        color: #666;
      }

      .user-address {
        margin-top: 14px;
        flex: 80%;

        label {
          margin-right: 30px;
        }
      }
    }

    .user-remark {
      height: 43px;
      line-height: 43px;
      background: #fffbf0;
      border: 1px solid #fbe396;
      border-radius: 4px;
      margin-top: 10px;
      padding: 6px;
      display: flex;
      align-items: center;

      div {
        display: inline-block;
        min-width: 53px;
        height: 32px;
        background: #fbe396;
        border-radius: 4px;
        text-align: center;
        line-height: 32px;
        color: #333;
        margin-right: 30px;
        // padding: 12px 6px;
      }

      span {
        color: #f2a402;
      }
    }

    .orderCancel {
      background: #ffffff;
      border: 1px solid #b6b6b6;

      div {
        padding: 0 10px;
        background-color: #e5e4e4;
      }

      span {
        color: #f56c6c;
      }
    }
  }

  .dish-info {
    // min-height: 180px;
    display: flex;
    flex-wrap: wrap;
    padding: 20px 40px;
    border-bottom: 1px solid #e7e6e6;

    .dish-label {
      color: #666;
    }

    .dish-list {
      flex: 80%;
      display: flex;
      flex-wrap: wrap;

      .dish-item {
        flex: 50%;
        margin-bottom: 14px;
        color: #333;

        .dish-num {
          margin-right: 51px;
        }
      }

      // .dish-item:nth-child(odd) {
      //   flex: 60%;
      // }
      // .dish-item:nth-child(even) {
      //   flex: 40%;
      // }
    }

    .dish-label {
      margin-right: 65px;
    }

    .dish-all-amount {
      flex: 1;
      padding-left: 92px;
      margin-top: 10px;

      label {
        color: #333333;
        font-weight: bold;
        margin-right: 5px;
      }

      span {
        color: #f56c6c;
      }
    }
  }
}

.order-bottom {
  .amount-info {
    // min-height: 180px;
    display: flex;
    flex-wrap: wrap;
    padding: 20px 40px;
    padding-bottom: 0px;

    .amount-label {
      color: #666;
      margin-right: 65px;
    }

    .amount-list {
      flex: 80%;
      display: flex;
      flex-wrap: wrap;
      color: #333;
      // height: 65px;
      .dish-amount,
      .package-amount,
      .pay-type {
        display: inline-block;
        width: 300px;
        margin-bottom: 14px;
        flex: 50%;
      }

      .send-amount,
      .all-amount,
      .pay-time {
        display: inline-block;
        flex: 50%;
        padding-left: 10%;
      }

      .package-amount {
        .amount-name {
          margin-right: 14px;
        }
      }

      .all-amount {
        .amount-name {
          margin-right: 24px;
        }

        .amount-price {
          color: #f56c6c;
        }
      }

      .send-amount {
        .amount-name {
          margin-right: 10px;
        }
      }
    }
  }
}
</style>
<style lang="scss">
.dashboard-container {
  .cancelTime {
    padding-left: 30px;
  }

  .orderTime {
    padding-left: 30px;
  }

  td.operate .cell {
    .before,
    .middle,
    .after {
      height: 39px;
      width: 48px;
    }
  }

  td.operate .cell,
  td.otherOperate .cell {
    display: flex;
    flex-wrap: nowrap;
    justify-content: center;
  }
}
</style>
