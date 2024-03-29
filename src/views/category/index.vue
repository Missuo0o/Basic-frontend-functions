<template>
  <div class="dashboard-container">
    <div class="container">
      <div class="tableBar"
           style="display: inline-block; width: 100%">
        <label style="margin-right: 10px">Name：</label>
        <el-input v-model="name"
                  clearable
                  style="width: 15%"
                  @clear="init"
                  @keyup.enter.native="init" />

        <label style="margin-right: 5px; margin-left: 20px">Type：</label>
        <el-select v-model="categoryType"
                   clearable
                   style="width: 15%"
                   @clear="init">
          <el-option v-for="item in options"
                     :key="item.value"
                     :label="item.label"
                     :value="item.value" />
        </el-select>

        <div style="float: right">
          <el-button class="continue"
                     type="primary"
                     @click="addClass('class')">
            + Add Dish Category
          </el-button>
          <el-button style="margin-left:20px"
                     type="primary"
                     @click="addClass('meal')">
            + Add Combo Category
          </el-button>
        </div>

        <el-button class="normal-btn continue"
                   @click="init(true)">
          Select
        </el-button>
      </div>
      <el-table v-if="tableData.length"
                :data="tableData"
                class="tableBox"
                stripe>
        <el-table-column label="Name"
                         prop="name" />
        <el-table-column label="Type"
                         prop="type">
          <template slot-scope="scope">
            <span>{{ scope.row.type == '1' ? 'Dish' : 'Combo' }}</span>
          </template>
        </el-table-column>

        <el-table-column label="Sort"
                         prop="sort" />
        <el-table-column label="Status">
          <template slot-scope="scope">
            <div :class="{ 'stop-use': String(scope.row.status) === '0' }"
                 class="tableColumn-status">
              {{ String(scope.row.status) === '0' ? 'Disable' : 'Enable' }}
            </div>
          </template>
        </el-table-column>
        <el-table-column label="Last Operation Time"
                         prop="updateTime" />
        <el-table-column align="center"
                         label="Operate"
                         width="250">
          <template slot-scope="scope">
            <el-button class="blueBug"
                       size="small"
                       type="text"
                       @click="editHandle(scope.row)">
              Update
            </el-button>
            <el-button class="delBut"
                       size="small"
                       type="text"
                       @click="deleteHandle(scope.row.id)">
              Delete
            </el-button>
            <el-button :class="{
                         blueBug: scope.row.status == '0',
                         delBut: scope.row.status != '0'
                       }"
                       class="non"
                       size="small"
                       type="text"
                       @click="statusHandle(scope.row)">
              {{ scope.row.status == '1' ? 'Disable' : 'Enable' }}
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <Empty v-else
             :is-search="isSearch" />
      <el-pagination v-if="counts > 10"
                     :page-size="pageSize"
                     :page-sizes="[10, 20, 30, 40]"
                     :total="counts"
                     class="pageList"
                     layout="total, sizes, prev, pager, next, jumper"
                     @size-change="handleSizeChange"
                     @current-change="handleCurrentChange" />
    </div>
    <el-dialog :before-close="handleClose"
               :title="classData.title"
               :visible.sync="classData.dialogVisible"
               width="30%">
      <el-form ref="classData"
               :model="classData"
               :rules="rules"
               class="demo-form-inline"
               label-width="100px">
        <el-form-item label="Name:"
                      prop="name">
          <el-input v-model="classData.name"
                    maxlength="20"
                    placeholder="Please enter the name" />
        </el-form-item>
        <el-form-item label="Sort:"
                      prop="sort">
          <el-input v-model="classData.sort"
                    placeholder="Please enter the sort" />
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button size="medium"
                   @click="
            ;(classData.dialogVisible = false), $refs.classData.resetFields()
                   ">Cancel</el-button>
        <el-button :class="{ continue: actionType === 'add' }"
                   size="medium"
                   type="primary"
                   @click="submitForm()">Submit</el-button>
        <el-button v-if="action != 'edit'"
                   size="medium"
                   type="primary"
                   @click="submitForm('go')">
          Submit and continue adding
        </el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import HeadLable from '@/components/HeadLable/index.vue'
import { addCategory, deleCategory, editCategory, enableOrDisableEmployee, getCategoryPage } from '@/api/category'
import Empty from '@/components/Empty/index.vue'

@Component({
  name: 'Category',
  components: {
    HeadLable,
    Empty
  }
})
export default class extends Vue {
  $refs!: {
    classData: any
  }
  private options: any = [
    {
      value: 1,
      label: 'Dish'
    },
    {
      value: 2,
      label: 'Combo'
    }
  ]
  private actionType: string = ''
  private id = ''
  private status = ''
  private categoryType: number = null
  private name: string = ''
  private action: string = ''
  private counts: number = 0
  private page: number = 1
  private pageSize: number = 10
  private tableData = []
  private type = ''
  private isSearch: boolean = false
  private classData: any = {
    title: 'Type',
    dialogVisible: false,
    type:'',
    categoryId: '',
    name: '',
    sort: ''
  }

  get rules() {
    return {
      name: [
        {
          required: true,
          trigger: 'blur',
          validator: (rule: any, value: string, callback: Function) => {
            // const reg = /[\u4e00-\u9fa5]/
            var reg = new RegExp('^[A-Za-z\u4e00-\u9fa5]+$')
            if (!value) {
              callback(new Error(this.classData.title + ' can not be empty'))
            } else if (value.length < 2) {
              callback(new Error('Please enter 2-20 characters'))
            } else {
              callback()
            }
          }
        }
      ],
      sort: [
        {
          required: true,
          trigger: 'blur',
          validator: (rule: any, value: string, callback: Function) => {
            if (value || String(value) === '0') {
              const reg = /^\d+$/
              if (!reg.test(value)) {
                callback(new Error('Please enter a number'))
              } else if (Number(value) > 99) {
                callback(new Error('Plase enter 0~99'))
              } else {
                callback()
              }
            } else {
              callback(new Error('Sorting cannot be empty'))
            }
          }
        }
      ]
    }
  }

  created() {
    this.init()
  }

  //数据提交
  submitForm(st: any) {
    if (this.action === 'add') {
      this.$refs.classData.validate((value: boolean) => {
        if (value) {
          addCategory({
            name: this.classData.name,
            type: this.classData.type,
            sort: this.classData.sort
          })
            .then(res => {
              if (res.data.code === 1) {
                this.$message.success('Success ！')
                this.$refs.classData.resetFields()
                if (!st) {
                  this.classData.dialogVisible = false
                }
                this.init()
              } else {
                this.$message.error(res.data.desc || res.data.msg)
              }
            })
            .catch(err => {
              this.$message.error('error:' + err.message)
            })
        }
      })
    } else {
      this.$refs.classData.validate((value: boolean) => {
        if (value) {
          editCategory({
            id: this.classData.id,
            type:this.classData.type,
            name: this.classData.name,
            sort: this.classData.sort
          })
            .then(res => {
              if (res.data.code === 1) {
                this.$message.success('Success ！')
                this.classData.dialogVisible = false
                this.$refs.classData.resetFields()
                this.init()
              } else {
                this.$message.error(res.data.desc || res.data.msg)
              }
            })
            .catch(err => {
              this.$message.error('error:' + err.message)
            })
        }
      })
    }
  }

  // 初始化信息
  private async init(isSearch?) {
    this.isSearch = isSearch
    await getCategoryPage({
      page: this.page,
      pageSize: this.pageSize,
      name: this.name ? this.name : undefined,
      type: this.categoryType ? this.categoryType : undefined
    })
      .then(res => {
        if (String(res.data.code) === '1') {
          this.tableData =
            res && res.data && res.data.data && res.data.data.records
          this.counts = Number(res.data.data.total)
        } else {
          this.$message.error(res.data.desc)
        }
      })
      .catch(err => {
        console.log(err, 'err')
        this.$message.error('error:' + err.message)
      })
  }

  // 添加
  private addClass(st: any) {
    if (st == 'class') {
      this.classData.title = 'Dish Category'
      this.classData.type = '1'
    } else {
      this.classData.title = 'Combo Category'
      this.classData.type = '2'
    }
    this.action = 'add'
    this.classData.name = ''
    this.classData.sort = ''
    this.classData.dialogVisible = true
    this.actionType = 'add'
  }

  // 修改
  private editHandle(dat: any) {
    this.classData.title = 'Update Category'
    this.action = 'edit'
    this.classData.name = dat.name
    this.classData.sort = dat.sort
    this.classData.type = dat.type
    this.classData.id = dat.id
    this.classData.dialogVisible = true
    this.actionType = 'edit'
  }

  // 关闭弹窗
  private handleClose(st: string) {
    console.log(this.$refs.classData, 'this.$refs.classData')
    this.classData.dialogVisible = false
    //对该表单项进行重置，将其值重置为初始值并移除校验结果
    this.$refs.classData.resetFields()
  }

  //状态修改
  private statusHandle(row: any) {
    this.id = row.id
    this.status = row.status
    this.$confirm('Confirm?', 'Comfirm', {
      confirmButtonText: 'Yes',
      cancelButtonText: 'No',
      type: 'warning',
      customClass: 'customClass'
    }).then(() => {
      enableOrDisableEmployee({ id: this.id, status: !this.status ? 1 : 0 })
        .then(res => {
          if (String(res.status) === '200') {
            this.$message.success('Success ！')
            this.init()
          }
        })
        .catch(err => {
          this.$message.error('error:' + err.message)
        })
    })
  }

  //删除
  private deleteHandle(id: any) {
    this.$confirm('Confirm?', 'Comfirm', {
      confirmButtonText: 'Yes',
      cancelButtonText: 'No',
      type: 'warning'
    }).then(() => {
      deleCategory(id)
        .then(res => {
          if (res.data.code === 1) {
            this.$message.success('Success ！')
            this.init()
          } else {
            this.$message.error(res.data.msg)
          }
        })
        .catch(err => {
          this.$message.error('error:' + err.message)
        })
    })
  }

  //分页
  private handleSizeChange(val: any) {
    this.pageSize = val
    this.init()
  }

  private handleCurrentChange(val: any) {
    this.page = val
    this.init()
  }
}
</script>
<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;

    .container {
      background: #fff;
      position: relative;
      z-index: 1;
      padding: 30px 28px;
      border-radius: 4px;

      .tableBar {
        display: flex;
        margin-bottom: 20px;
        justify-content: space-between;
      }

      .tableBox {
        width: 100%;
        border: 1px solid $gray-5;
        border-bottom: 0;
      }

      .pageList {
        text-align: center;
        margin-top: 30px;
      }

      //查询黑色按钮样式
      .normal-btn {
        background: #333333;
        color: white;
        margin-left: 20px;
      }
    }
  }
}
</style>
<style lang='scss'>
// .customClass {
//   .el-button--primary {
//     background-color: #ffc200 !important ;
//   }
// }
</style>
