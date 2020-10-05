<template>
  <div>
    <el-form label-width="100px" :model="skuInfo">
      <el-form-item label="SPU名称">{{spu.spuName}}</el-form-item>
      <el-form-item label="SKU名称">
        <el-input placeholder="SKU名称" v-model="skuInfo.skuName"></el-input>
      </el-form-item>
      <el-form-item label="价格(元)">
        <el-input placeholder="价格" type="number" v-model="skuInfo.price"></el-input>
      </el-form-item>
      <el-form-item label="重量(KG)">
        <el-input placeholder="重量" type="number" v-model="skuInfo.weight"></el-input>
      </el-form-item>
      <el-form-item label="规格描述">
        <el-input placeholder="规格描述" type="textarea" rows="4" v-model="skuInfo.skuDesc"></el-input>
      </el-form-item>
      <el-form-item label="平台属性">
        <el-form :inline="true" label-width="100px">
          <el-form-item
            :label="attrInfo.attrName"
            v-for="(attrInfo, index) in attrInfoList"
            :key="attrInfo.id"
          >
            <el-select placeholder="请输入" v-model="attrInfo.attrIdAttrValueId">
              <el-option
                :label="attrValue.valueName"
                :value="`${attrInfo.id}:${attrValue.id}`"
                v-for="(attrValue, index) in attrInfo.attrValueList"
                :key="attrValue.id"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>

      <el-form-item label="销售属性">
        <el-form :inline="true" label-width="100px">
          <el-form-item
            :label="spuSaleAttr.saleAttrName"
            v-for="(spuSaleAttr, index) in spuSaleAttrList"
            :key="spuSaleAttr.id"
          >
            <el-select placeholder="请输入" v-model="spuSaleAttr.saleAttrValueId">
              <el-option
                :label="spuSaleAttrValue.saleAttrValueName"
                :value="spuSaleAttrValue.id"
                v-for="(spuSaleAttrValue, index) in spuSaleAttr.spuSaleAttrValueList"
                :key="spuSaleAttrValue.id"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>

      <el-form-item label="图片列表">
        <el-table border style="width: 100%" :data="spuImageList" @selection-change="handleSelectionChange">
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column label="图片" width="width">
            <template slot-scope="{row,$index}">
              <img :src="row.imgUrl" alt style="width:80px;height:100px" />
            </template>
          </el-table-column>
          <el-table-column prop="imgName" label="名称" width="width"></el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{row,$index}">
              <el-button v-if="row.isDefault === '0'" type="primary" size="mini" @click="setDefault(row)">设为默认</el-button>
              <el-tag type="success" v-else>默认</el-tag>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="save">保存</el-button>
        <el-button @click="back">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: 'SkuForm',

  data(){
    return{
      spu:{},
      attrInfoList:[],
      spuSaleAttrList:[],
      spuImageList:[],
      imgList:[],
      skuInfo:{
        category3Id: 0,
        spuId: 0,
        tmId: 0,
        price: 0,
        skuDesc: "",
        weight: "",
        skuDefaultImg: "",
        skuAttrValueList: [],
        skuImageList: "",
        skuAttrValueList: [],
      }
    }
  },

  methods:{

    back(){
      this.$emit('update:visible',false)
      this.resetData()
    },

    resetData(){
      this.spu = {}
      this.attrInfoList = []
      this.spuSaleAttrList = []
      this.spuImageList = []

      this.imgList = []

      this.skuInfo = {
        category3Id: 0,
        spuId: 0,
        tmId: 0,
        price: 0,
        skuDesc: "",
        skuName: "",
        weight: "",

        skuDefaultImg: "",
        skuAttrValueList: [],
        skuImageList: [],
        skuSaleAttrValueList: [],
      }
    },

    async save(){
      let {attrInfoList,spuSaleAttrList,imgList,skuInfo} = this

      skuInfo.skuImageList = imgList.map(item => {
        return{
          imgName:item.imgName,
          imgUrl:item.imgUrl,
          isDefault:item.isDefault,
          spuImgId:item.id
        }
      })

      attrInfoList.forEach(item => {
        if(item.attrIdAttrValueId) {
          let [attrId,valueId] = item.attrIdAttrValueId.split(':')
          skuInfo.skuAttrValueList.push({
            attrId,
            valueId
          })
        }
      })

      skuInfo.skuSaleAttrValueList = spuSaleAttrList.reduce((pre,item) => {
        if(item.saleAttrValueId){
          pre.push({
            saleAttrValueId:item.saleAttrValueId
          })
        }
        return pre
      },[])

      const result = await this.$API.sku.addUpdate(skuInfo)
      if(result.code === 200){
        this.$message.success('保存sku成功')
        this.$emit('update:visble',false)
        this.resetData()
      }else{
        this.$message.error('保存sku失败')
      }
    },

    serDefault(){
      this.spuImageList.forEach(item => item.isDefault = '0')
      row.isDefault = '1'
      this.skuInfo.skuDefaultImg = row.imgUrl
    },

    handleSelectionChange(val){
      this,imgList = val
    },

    async initAddSkuData(spu, category1Id, category2Id){
      this.spu = spu;

       this.skuInfo.category3Id = spu.category3Id
       this.skuInfo.tmId = spu.tmId
       this.skuInfo.spuId = spu.id


       const promise1 = this.$API.attr.getList(
         category1Id,
         category2Id,
         spu.category3Id
       );

       const promise2 = this.$API.sku.getSpuSaleAttrList(spu.id);
       const promise3 = this.$API.sku.getSpuImageList(spu.id);


       const result = await Promise.all([promise1, promise2, promise3]);
       this.attrInfoList = result[0].data;
       this.spuSaleAttrList = result[1].data;
       let spuImageList = result[2].data;

       spuImageList.forEach(item => {
         item.isDefault = '0'
       })

       this.spuImageList = spuImageList
    }
  }
}
</script>

<style scoped>

</style>
