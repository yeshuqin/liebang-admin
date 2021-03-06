<template>
  <div class="_list">
    <el-form :inline="true" size="small" :model="formInline" label-width="100px">
      <el-row>
        <el-col :span="20">
          <el-form-item label="商品ID">
            <el-input v-model.trim="formInline.id" clearable placeholder="请输入商品ID" />
          </el-form-item>
          <el-form-item label="一级类别">
            <el-select v-model="formInline.cateId" filterable clearable placeholder="请选择">
              <el-option
                v-for="item in cateList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="服务类型">
            <el-input v-model.trim="formInline.name" clearable placeholder="请输入商品名称" />
          </el-form-item>
          <el-form-item label="商品状态">
            <el-select v-model="formInline.saleStatus" filterable clearable placeholder="请选择">
              <el-option
                v-for="item in statusItems"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="上架时间">
            <el-date-picker
              v-model="saleTime"
              value-format="timestamp"
              type="datetimerange"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
            />
          </el-form-item>
        </el-col>
        <el-col :span="4">
          <el-form-item>
            <el-button type="primary" size="small" icon="el-icon-search" @click="handleSearch">搜索</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" size="small" icon="el-icon-plus" @click="handleAddSpu">新增</el-button>
          </el-form-item>
          <br>
          <el-form-item>
            <el-button type="defalut" @click="handleBatchStatus(1)">批量上架</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="defalut" @click="handleBatchStatus(0)">批量下架</el-button>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <div class="table_wrapper">
      <tl-table
        :table="dataTable"
        @onHandleSelectionChange="onHandleSelectionChange"
        @sizeChange="sizeChange"
        @pageChange="pageChange"
        @handleDel="handleDel"
        @handleEdit="handleEdit"
        @handleSKUList="handleSKUList"
        @handleStatus="handleStatus"
      >
        <template slot="primaryPic" slot-scope="props">
          <span v-if="!props.obj.row.primaryPic">暂无图片</span>
          <img v-else :src="props.obj.row.primaryPic" class="picUrl" @click="handlePreview(props.obj.row)" alt="">
        </template>
        <template slot="saleStatus" slot-scope="props">
          <span>{{ props.obj.row.saleStatus === 0 ? '未上架' : '已上架' }}</span>
        </template>
        <template slot="saleTime" slot-scope="props">
          <span>{{ props.obj.row.saleTime | timestampToTime }}</span>
        </template>
        <template slot="minPrice" slot-scope="props">
          <span>¥{{ props.obj.row.minPrice | filterMoney }} - {{ props.obj.row.maxPrice | filterMoney}}</span>
        </template>
        <template slot="handleStatus" slot-scope="props">
          <span v-if="props.obj.row.saleStatus === 0" class="link_btn" @click="handleStatus(props.obj.row)">上架</span>
          <span v-else class="link_btn" @click="handleStatus(props.obj.row)">下架</span>
        </template>
      </tl-table>
    </div>
    <!-- sku列表 -->
    <el-dialog :title="`${spuName} SKU列表`" :visible.sync="showSkuDialog" custom-class="sku_dialog" width="1200px" center>
      <tl-table
        :showpagination="false"
        :table="skuDataTable"
        @handleDel="handleDelSKU"
      >
        <template slot="saleStatus" slot-scope="props">
          <span>{{ props.obj.row.saleStatus === 0 ? '未上架' : '已上架' }}</span>
        </template>
        <template slot="attribute" slot-scope="props">
          <span v-if="props.obj.row.attribute">{{ getAttribute(props.obj.row.attribute) }}</span>
        </template>
        <template slot="costPrice" slot-scope="props">
          <span>¥{{ props.obj.row.costPrice | filterMoney }}</span>
        </template>
        <template slot="salePrice" slot-scope="props">
          <span>¥{{ props.obj.row.salePrice | filterMoney }}</span>
        </template>
        <template slot="marketPrice" slot-scope="props">
          <span>¥{{ props.obj.row.marketPrice | filterMoney }}</span>
        </template>
         <template slot="name" slot-scope="props">
          <span v-if="props.obj.row.attribute">{{ spuName }} {{getAttribute(props.obj.row.attribute) }}</span>
        </template>
        <template slot="handleStatus" slot-scope="props">
          <span v-if="props.obj.row.saleStatus === 0" class="link_btn" @click="handleSKUStatus(props.obj.row)">上架</span>
          <span v-else class="link_btn" @click="handleSKUStatus(props.obj.row)">下架</span>
        </template>
      </tl-table>
      <div slot="footer" class="dialog-footer">
        <!-- <el-button type="primary" size="small"></el-button> -->
        <el-button size="small" @click="showSkuDialog = false">关 闭</el-button>
      </div>
    </el-dialog>
    <ImgDialog :showViewImgDialog.sync="showViewImgDialog" :imgSrc="imgSrc"></ImgDialog>
  </div>
</template>

<script>
import tlTable from '@/components/BaseTable/tlTable'
import ImgDialog from '@/components/ImgDialog/ImgDialog'

export default {
  name: 'GoodsList',
  components: {
    tlTable,
    ImgDialog
  },
  data() {
    return {
      formInline: {
        id: '',
        name: '',
        cateId: '',
        saleStatus: '',
        saleTimeStart: '',
        saleTimeEnd: ''
      },
      statusItems: [
        {
          value: 0,
          label: '未上架'
        },

        {
          value: 1,
          label: '已上架'
        }
      ],
      dataTable: {
        hasSelect: true,
        hasExpand: false,
        loading: false,
        page: 1,
        size: 20,
        total: 0,
        expands: [],
        tr: [
          {
            label: '商品ID',
            prop: 'id',
            init: '-'
          },
          {
            label: '一级类别',
            prop: 'cateName',
            init: '-'
          },
          {
            label: '服务类型',
            prop: 'name',
            init: '-'
          },
          {
            label: '主图',
            prop: 'primaryPic',
            width:"120px",
            slot: true,
            init: '—'
          },
          {
            label: '最低-最高销售价',
            prop: 'minPrice',
            width: '130px',
            slot: true,
            init: '—'
          },
          {
            label: '标签',
            prop: 'tags',
            init: '—'
          },
          {
            label: '上架时间',
            prop: 'saleTime',
            width: '100px',
            slot: true,
            init: '—'
          },
          {
            label: '创建时间',
            prop: 'createTime',
            width: '100px',
            init: '—'
          },
          {
            label: '商品状态',
            prop: 'saleStatus',
            slot: true,
            init: '—'
          }
        ],
        data: [],
        operation: {
          width: '140',
          data: [
            {
              label: '编辑',
              Fun: 'handleEdit'
            },
            {
              label: 'SKU列表',
              Fun: 'handleSKUList'
            },
            {
              slot: true,
              Fun: 'handleStatus'
            },
            {
              label: '删除',
              Fun: 'handleDel'
            }
          ]
        }
      },
      skuDataTable: {
        hasSelect: false,
        hasExpand: false,
        loading: false,
        page: 1,
        size: 50,
        total: 0,
        expands: [],
        tr: [
          {
            label: 'SKUID',
            prop: 'id',
            init: '-'
          },
          {
            label: 'SKU名称',
            prop: 'name',
            width: '280px',
            slot: true,
            init: '—'
          },
          {
            label: '属性',
            prop: 'attribute',
            width: '200px',
            slot: true,
            init: '—'
          },
          {
            label: '成本价',
            prop: 'costPrice',
            slot: true,
            init: '—'
          },
          {
            label: '销售价',
            prop: 'salePrice',
            slot: true,
            init: '—'
          },
          {
            label: '市场价',
            prop: 'marketPrice',
            slot: true,
            init: '—'
          },
          {
            label: '商品状态',
            prop: 'saleStatus',
            slot: true,
            init: '—'
          }
        ],
        data: [],
        operation: {
          width: '100',
          data: [
            {
              slot: true,
              Fun: 'handleStatus'
            },
            {
              label: '删除',
              Fun: 'handleDel'
            }
          ]
        }
      },
      spuName: '',
      spuId: '',
      saleTime: [],
      checkBox: [],
      cateList: [],
      showSkuDialog: false,
      showViewImgDialog: false,
      imgSrc: ''
    }
  },
  created() {
    this.getInfor()
    this.getCate()
  },
  methods: {
    getInfor() {
      if (this.saleTime) {
        this.formInline.saleTimeStart = this.saleTime[0] / 1000
        this.formInline.saleTimeEnd = this.saleTime[1] / 1000
      } else {
        this.formInline.saleTimeStart = ''
        this.formInline.saleTimeEnd = ''
      }
      var url = `${this.$api.spu}/page`
      var params = Object.assign({}, {
        current: this.dataTable.page,
        size: this.dataTable.size
      }, this.formInline)
      this.dataTable.loading = true
      this.$http.send(url, params, 'post').then(res => {
        if (res.data) {
          this.dataTable.data = res.data.records
          this.dataTable.total = res.data.total
        }
        this.dataTable.loading = false
      }).catch(res => {
        this.dataTable.loading = false
      })
    },
    getCate() {
      this.$http.send(this.$api.addCate, {}, 'get').then(res => {
        if (res.data) {
          this.cateList = res.data
        }
      }).catch(res => {
      })
    },
    onHandleSelectionChange(val) {
      this.checkBox = val.map(item => {
        return item.id
      })
    },
    handleSearch() {
      this.dataTable.page = 1
      this.getInfor()
    },
    handleAddSpu(row) { // 新增spu
      this.$router.push({ name: 'GoodsSpuAdd'})
    },
    handleEdit(row) { // 编辑spu
      this.$router.push({ name: 'GoodsSpuAudit', query: { id: row.id }})
    },
    handleDel(row) {
      this.$confirm('此操作将删除商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        var url = `${this.$api.spu}/${row.id}`
        this.$http.send(url, {}, 'delete').then(res => {
          this.$message.success('操作成功~')
          this.getInfor()
        }).catch(res => {
          // this.$message.error(res.msg)
        })
      })
    },
    handleDelSKU(row) {
      this.$confirm('此操作将删除商品SKU, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        var url = `${this.$api.sku}/${row.id}`
        this.$http.send(url, {}, 'delete').then(res => {
          this.$message.success('操作成功~')
          this.getSkuList()
        }).catch(res => {
          // this.$message.error(res.msg)
        })
      })
    },
    handleStatus(row) {
      this.$confirm(`此操作将${row.saleStatus === 0 ? '上架' : '下架'}[${row.name}]商品, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        var status = row.saleStatus === 0 ? 1 : 0
        var url = `${this.$api.spu}/setting`
        this.$http.send(url, {
          spuList: [row.id],
          operation: status
        }, 'patch').then(res => {
          this.$message.success('操作成功~')
          this.getInfor()
        }).catch(res => {
          // this.$message.error(res.msg)
        })
      })
    },
    handleSKUStatus(row) {
      this.$confirm(`此操作将${row.saleStatus === 0 ? '上架' : '下架'}[${row.id}]商品SKU, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        var status = row.saleStatus === 0 ? 1 : 0
        var url = `${this.$api.sku}/${row.id}/setting/${status}`
        this.$http.send(url, {}, 'patch').then(res => {
          this.$message.success('操作成功~')
          this.getSkuList()
        }).catch(res => {
          // this.$message.error(res.msg)
        })
      })
    },
    handleSKUList(row) { //查看SKU列表
      this.showSkuDialog = true
      this.spuName = row.name
      this.spuId = row.id
      this.getSkuList()
    },
    getSkuList() {
      var url = `${this.$api.sku}/spu/${this.spuId}`
      this.$http.send(url, {}, 'get').then(res => {
        this.skuDataTable.data = res.data
        // this.skuDataTable.data.forEach(item => {
          // item.costPrice = item.costPrice / 100
        // })
      }).catch(res => {
        // this.$message.error(res.msg)
      })
    },
    getAttribute(str) {
      var obj = JSON.parse(str)
      var arr = []
      for (var i in obj) {
        arr.push(`${[i]}：${obj[i]}`)
      }
      var key = arr.join(',')
      return key
    },
    handleBatchStatus(type) {
      if (this.checkBox.length === 0) {
        this.$message.error('请至少选择一项~')
        return
      }
      this.$confirm(`此操作将批量${type === 1 ? '上架' : '下架'}商品, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        var url = `${this.$api.spu}/setting`
        this.$http.send(url, {
          spuList: this.checkBox,
          operation: type
        }, 'patch').then(res => {
          this.$message.success('操作成功~')
          this.getInfor()
        }).catch(res => {
          // this.$message.error(res.msg)
        })
      })
    },
    pageChange(page) {
      this.dataTable.page = page
      this.getInfor()
    },
    sizeChange(size) {
      this.dataTable.size = size
      this.getInfor()
    },
    handlePreview (row) {
      this.showViewImgDialog = true
      this.imgSrc = row.primaryPic
    }
  }
}
</script>

<style lang="scss" scoped>
.add_dialog {
  .el-input, .el-select {
    width: 70%;
  }
  .line {
    text-align: center;
  }
  .el-upload__tip {
    margin-left: 20px;
  }
}
.download_checkbox {
  .el-checkbox {
    width: 50%;
    font-size: 16px !important;
    margin-right: 0;
    margin-bottom: 10px;
    text-align: center;
  }
}
.title {
  font-size: 18px;
  text-align: center;
}
</style>
