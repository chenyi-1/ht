<template>
  <div>
    <el-card>
   <el-table data="" border style="width: 100%">
    <el-table-column prop="prop" label="序号" width="80" align="center">
    </el-table-column>
    <el-table-column prop="prop" label="名称" width="180">
    </el-table-column>
    <el-table-column  prop="prop" label="描叙" width="">
    </el-table-column>
    <el-table-column  prop="prop" label="默认图片" width="150" align="center">
    </el-table-column>
    <el-table-column  prop="prop" label="重量(KG)" >
    </el-table-column>
    <el-table-column  prop="prop" label="价格(元)" width="80">
    </el-table-column>
    <el-table-column  prop="prop" label="操作" width="180" align="center">
    </el-table-column>
  </el-table>
    </el-card>

      <el-pagination
      style="text-align:center"
      @size-change="handleSizeChange"
      @current-change="getTrademarkList"
      :current-page="page"
      :page-sizes="[3, 5, 10]"
      :page-size="limit"
      layout=" prev, pager, next, jumper,->, sizes,total"
      :total="total"
    ></el-pagination>

  </div>
</template>

<script>
export default {
  name: 'Sku',
  methods:{
    async showSkuInfo(){
      this.isShowSkuInfo = true
      const result = await this.$API.sku.get(id)
      this.skuInfo = result.data
    },

    async getSkuList(page = 1) {
      this.page = page
      this.loading = true
      const result = await this.$API.sku.getList(this.page, this.limit)
      this.skuList = result.data.records
      this.total = result.data.total
      this.loading = false
    }
  }
}
</script>


