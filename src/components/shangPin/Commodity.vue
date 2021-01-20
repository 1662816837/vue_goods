<template>
  <div>
    <h1>商品管理</h1>
    <el-steps :active="active" finish-status="success" simple style="margin-top: 20px">
      <el-step title="步骤 1"></el-step>
      <el-step title="步骤 2"></el-step>
      <el-step title="步骤 3"></el-step>
    </el-steps>
    <div class="info" v-if="active==1">
      <el-form :model="productInfoForm" :rules="rules" ref="productInfoForm"  label-width="120px" style="width: 600px" size="small">
        <!--<el-form-item label="商品分类：" prop="productCategoryId">

          &lt;!&ndash;<el-cascader
            v-model="selectProductCateValue"
            :options="productCateOptions">
          </el-cascader>&ndash;&gt;
        </el-form-item>-->
        <el-form-item label="商品名称：" prop="name">
          <el-input v-model="productInfoForm.name"></el-input>
        </el-form-item>
        <el-form-item label="副标题：" prop="subTitle">
          <el-input v-model="productInfoForm.subTitle"></el-input>
        </el-form-item>
        <el-form-item label="商品品牌：" prop="brandId">
          <el-select
            v-model="productInfoForm.brandId"
            placeholder="请选择品牌">
            <el-option
              v-for="item in brandDatas"
              :key="item.id"
              :label="item.name"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品介绍：">
          <el-input
            :autoSize="true"
            v-model="productInfoForm.description"
            type="textarea"
            placeholder="请输入内容"></el-input>
        </el-form-item>
        <el-form-item label="商品售价：">
          <el-input v-model="productInfoForm.price"></el-input>
        </el-form-item>
        <el-form-item label="商品库存：">
          <el-input v-model="productInfoForm.stock"></el-input>
        </el-form-item>
        <el-form-item label="排序">
          <el-input v-model="productInfoForm.sort"></el-input>
        </el-form-item>
        <el-form-item style="text-align: center">
          <!--<el-button type="primary" size="medium" @click="next">下一步，填写商品促销</el-button>-->
        </el-form-item>
      </el-form>
    </div>

    <div class="info" v-if="active==2">
      <h1>展示页2</h1>
      <el-form :model="productForm2" label-width="100px" class="demo-ruleForm">

        <el-form-item label="分类" prop="typeId">
          <!--  改变 获取属性数据  -->
          <el-select v-model="productForm2.typeId" placeholder="请选择" @change="getAttrData">
            <el-option v-for="item in TypeDatas" :key="item.id" :label="item.name" :value="item.id"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item v-if="SKUData.length>0" label="商品规格" prop="name">

          <el-form-item v-for="a in  SKUData" :key="a.id" :label="a.nameCH">

            <el-checkbox-group v-model="a.ckValues" v-if="a.type==2" @change="skuChange">
              <el-checkbox v-for="b in a.values" :key="b.id" :label="b.nameCH" name="type"></el-checkbox>
            </el-checkbox-group>

          </el-form-item>

        </el-form-item>



        <el-table
          v-if="tableShow"
          :data="tableData"
          style="width: 100%">

          <el-table-column v-for="c in cols" :key="c.id" :label="c.nameCH" :prop="c.name">
          </el-table-column>

          <el-table-column
            label="库存"
            width="180">

            <template slot-scope="scope">
              <el-input/>
            </template>

          </el-table-column>
          <el-table-column
            label="价格"
            width="180">
            <template slot-scope="scope">
              <el-input/>
            </template>
          </el-table-column>
        </el-table>




        <el-form-item v-if="attData.length>0" label="商品参数" prop="name">

          <el-form-item v-for="a in  attData" :key="a.id" :label="a.nameCH">

            <!--  0 下拉框     1 单选框      2  复选框   3  输入框  -->
            <el-input v-if="a.type==3">

            </el-input>

            <el-select v-if="a.type==0" v-model="ooa" placeholder="请选择">
              <el-option v-for="b in a.values" :key="b.id" :label="b.nameCH" :value="b.id"></el-option>
            </el-select>

            <el-radio-group v-model="radioaaa" v-if="a.type==1">
              <el-radio v-for="b in a.values" :key="b.id" :label="b.nameCH"></el-radio>
            </el-radio-group>

            <el-checkbox-group v-model="ss" v-if="a.type==2">
              <el-checkbox v-for="b in a.values" :key="b.id" :label="b.nameCH" name="type"></el-checkbox>
            </el-checkbox-group>

          </el-form-item>

        </el-form-item>


      </el-form>
    </div>
    <div class="info" v-if="active==3">
      <h1>展示页3</h1>
    </div>
    <el-button type="primary" @click="onTop">上一步</el-button>
    <el-button type="primary" @click="next">下一步</el-button>

  </div>
</template>

<script>
  export default {
    name: "Commodity",
    data() {
      return {
        rules:{
          name: [
            { required: true, message: '请输入活动名称', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ]
        },
        active: 1,
        radioaaa:"",
        brandDatas: [],
        TypeData: [],
        TypeDatas: [],
        tableaa:"",
        tableData:[],

        cols:[],//表动态列头
        ifif:"",
        ooa: "",
        ss: [],
        typeName: "",
        productInfoForm: {
          name: "",
          subTitle: "",
          productSn: "",
          price: "",
          originalPrice: "",
          stock: "",
          unit: "",
          weight: "",
          sort: "",
          description: "",
          brandId: ""
        },
        productForm2: {},
        attData: [],   //非sku的属性数据
        SKUData: [], //sku属性数据

        tableShow: false


      }
    }, created: function () {
      this.getBrandData();
    },
    methods: {
      discarts: function () {
        //笛卡尔积
        var twodDscartes = function (a, b) {
          var ret = [];
          for (var i = 0; i < a.length; i++) {
            for (var j = 0; j < b.length; j++) {
              ret.push(ft(a[i], b[j]));
            }
          }
          return ret;
        }
        var ft = function (a, b) {
          if (!(a instanceof Array))
            a = [a];
          var ret = a.slice(0);
          ret.push(b);
          return ret;
        }
        //多个一起做笛卡尔积
        return (function (data) {
          var len = data.length;
          if (len == 0)
            return [];
          else if (len == 1)
            return data[0];
          else {
            var r = data[0];
            for (var i = 1; i < len; i++) {
              r = twodDscartes(r, data[i]);
            }
            return r;
          }
        })(arguments.length > 1 ? arguments : arguments[0]);
      },

      //监听sku属性 改变事件
      skuChange: function () {

        //清空动态列头
        this.cols=[];
        this.tableData=[];


        console.log(this.SKUData);
        //判断是否要生成笛卡尔积
        let flag = true;
        var ccc = [];
        for (let i = 0; i < this.SKUData.length; i++) {
          if (this.SKUData[i].ckValues.length == 0) {
            flag = false;
            break;
          } else {
           /* this.cols.push()*/
            ccc.push(this.SKUData[i].ckValues);
            this.cols.push(this.SKUData[i])
          }
        }
        if (flag == true) {


          var iiu = this.discarts(ccc);
          this.tableShow = flag;
          for (var i = 0; i <iiu.length; i++) {
            let valuesAttr=iiu[i];
            let  tableValue={};
            var lie = iiu[i];
            for(var j = 0 ; j <lie.length ; j++){
              var key = this.cols[j].name;
              tableValue[key]=valuesAttr[j]
            }
            this.tableData.push(tableValue)
          }

        }else{
          this.tableShow = false;
        }

      },

      next: function () {
        this.queryTypeData();
        if (this.active++ > 2) this.active = 1;
      },
      queryTypeData: function () {
        this.$ajax.get("http://localhost:8080/GoodsController/getData").then(res => {
          console.log(res)
          this.TypeData = res.data.data;
          this.getTypeDatas()
          //遍历所有的子节点
          for (var i = 0; i < this.TypeDatas.length; i++) {
            //定义的全局属性 用来存储我们拼接的底层信息
            this.typeName = "";
            //处理子节点的name属性
            this.chandleName(this.TypeDatas[i]);
            this.TypeDatas[i].name = this.typeName.split("/").reverse().join("/").substring(0, this.typeName.length - 1);
          }
        }).catch(err => console.log(err))
      },
      getTypeDatas() {
        for (var i = 0; i < this.TypeData.length; i++) {
          var nnn = this.TypeData[i];
          //判断所有底层节点 并且赋值到下拉框
          this.isParent(nnn)
        }
        console.log(this.TypeDatas)
      },
      isParent: function (datas) {
        var ccc = true;
        for (var j = 0; j < this.TypeData.length; j++) {
          if (datas.id == this.TypeData[j].pid) {
            ccc = false;
            break;
          }
        }
        if (ccc == true) {
          this.TypeDatas.push(datas)
        }
      },
      chandleName: function (node) {
        if (node.pid != 0) {
          this.typeName += "/" + node.name;
          //判断我们传过来的值存不存在父节点
          for (var i = 0; i < this.TypeData.length; i++) {
            if (node.pid == this.TypeData[i].id) {
              this.chandleName(this.TypeData[i]);
              break;
            }
          }
        } else {
          this.typeName += "/" + node.name;
        }
      },
      getAttrData: function (typeId) {
        this.$ajax.post("http://localhost:8080/AttributeController/queryDataBuTypeId?typeId=" + typeId).then(res => {
          // 所有的属性数据
          this.SKUData = [];
          this.attData = [];
          let attrDatas = res.data.data;
          //判断分类是否有数据   更新 参数和规格
          if (attrDatas.length > 0) {
            //初始化  attData      SKUData
            for (let i = 0; i < attrDatas.length; i++) {
              //判断是否为sku属性
              if (attrDatas[i].isSKU == 1) {

                if (attrDatas[i].type != 3) {
                  //发起请求 查询此属性对应的选项值
                  this.$ajax.post("http://localhost:8080/AttributeValueController/queryDate?attId=" + attrDatas[i].id).then(res => {
                    console.log(res)
                    attrDatas[i].values = res.data.data;
                    this.attData.push(attrDatas[i]);
                  })
                } else {
                  this.attData.push(attrDatas[i]);
                }

              } else {
                if (attrDatas[i].type != 3) {
                  //发起请求 查询此属性对应的选项值
                  this.$ajax.post("http://localhost:8080/AttributeValueController/queryDate?attId=" + attrDatas[i].id).then(res => {
                    attrDatas[i].values = res.data.data;
                    attrDatas[i].ckValues = [];
                    this.SKUData.push(attrDatas[i]);
                  })
                } else {
                  attrDatas[i].ckValues = [];
                  this.SKUData.push(attrDatas[i]);
                }

              }
            }
            console.log(this.attData)
          }

        }).catch(err => console.log(err))
      },
      onTop: function () {
        if (this.active-- < 2) this.active = 1;
      },
      getBrandData: function () {
        this.$ajax.post("http://localhost:8080/BrandController/queryDataAll").then(res => {
          console.log(res)
          this.brandDatas = res.data.data;
        }).catch(err => console.log(err))
      }
    }


  }
</script>

<style scoped>

</style>
