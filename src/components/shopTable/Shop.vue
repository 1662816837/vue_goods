<template>
  <div>
    <h1>商品展示页面</h1>
    <!-------------------------------------条件查询展示----------------------------------------------------->
    <div id="searchDiv">
      <el-form :inline="true" :model="queryForm" class="demo-form-inline">

        <el-form-item label="名称">
          <el-input v-model="queryForm.name" placeholder="名称"></el-input>
        </el-form-item>

        <el-form-item>
          <el-button type="primary">查询</el-button>
          <!--<el-button type="success" @click="xinzneg">新增</el-button>-->
        </el-form-item>
      </el-form>
    </div>
    <!-------------------------------------数据表格展示------------------------------------------------------------------------>
    <div id="carTable">
      <el-table
        :data="TableData"
        style="width: 100%">

        <el-table-column
          prop="id"
          label="序号"
          width="80">
        </el-table-column>

        <el-table-column
          prop="name"
          label="名称"
          width="80">
        </el-table-column>

        <el-table-column
          prop="imgPath"
          label="图片">
          <template slot-scope="scope">
            <img width="50px" :src="scope.row.imgPath"/>
          </template>
        </el-table-column>

        <el-table-column
          prop="title"
          label="标题"
          width="80">
        </el-table-column>

        <el-table-column
          prop="bandId"
          label="品牌"
          width="80"
          :formatter="SpellBrand"
        >
        </el-table-column>

        <el-table-column
          prop="typeId"
          label="分类"
          width="80"
          :formatter="changetypeId"
        >
        </el-table-column>

        <el-table-column
          prop="productdecs"
          label="简介"
          width="80">
        </el-table-column>

        <el-table-column
          prop="price"
          label="价格"
          width="80">
        </el-table-column>

        <el-table-column
          prop="stocks"
          label="库存"
          width="80">
        </el-table-column>

        <el-table-column
          prop="createDate"
          label="创建时间">
        </el-table-column>

        <el-table-column
          prop="updateDate"
          label="修改时间">
        </el-table-column>

        <el-table-column
          prop="id"
          label="操作"
          width="200px">

          <template slot-scope="scope">
            <el-button type="success" icon="el-icon-edit" @click="toUpdateTable(scope.row.id)" circle></el-button>
            <el-button type="success" @click="upSkuTable(scope.row)">属性值维护</el-button>
            <!--<el-button type="info" @click="deleteBrand(scope.row.id)" icon="el-icon-delete" circle></el-button>-->
          </template>
        </el-table-column>

      </el-table>

      <el-pagination
        @current-change="handleCurrentChange"
        @size-change="handleSizeChange"
        :page-sizes="sizes"
        :page-size="size"
        layout="total, sizes, prev, pager, next, jumper"
        :total="count">
      </el-pagination>
    </div>
    <!----------------------------------------------------表格修改模板------------------------------------------->
    <el-dialog title="商品信息" :visible.sync="updateTableData">
      <el-form :model="updateData" ref="addForm" label-width="100px">
        <el-form-item label="商品名称：" prop="name">
          <el-input v-model="updateData.name"></el-input>
        </el-form-item>
        <el-form-item label="副标题：" prop="subTitle">
          <el-input v-model="updateData.title"></el-input>
        </el-form-item>
        <el-form-item label="商品品牌：" prop="brandId">
          <el-select
            v-model="updateData.bandId"
            placeholder="请选择品牌">
            <el-option
              v-for="item in brandData"
              :key="item.id"
              :label="item.name"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品介绍：">
          <el-input
            :autoSize="true"
            v-model="updateData.productdecs"
            type="textarea"
            placeholder="请输入内容"></el-input>
        </el-form-item>
        <el-form-item label="商品售价：">
          <el-input v-model="updateData.price"></el-input>
        </el-form-item>

        <el-form-item label="图片">
          <img width="50px" :src="updateData.imgPath"/>
          <el-upload
            class="upload-demo"
            action="http://localhost:8080/BrandController/uploadFile"
            :on-success="uploadImage"
            name="file"
            list-type="picture">
            <el-button size="small" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>
        </el-form-item>

        <el-form-item label="商品库存：">
          <el-input v-model="updateData.stocks"></el-input>
        </el-form-item>
        <el-form-item label="排序">
          <el-input v-model="updateData.sortNum"></el-input>
        </el-form-item>


        <el-button @click="updateTableData = false">取 消</el-button>
        <el-button type="primary" @click="updateForm">确 定</el-button>

      </el-form>
    </el-dialog>
    <!----------------------------------------SKU修改模板------------------------------------------->
    <el-dialog title="商品信息" :visible.sync="SkuDataTable">

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
          :data="skuTableData"
          style="width: 100%">

          <el-table-column v-for="c in cols" :key="c.id" :label="c.nameCH" :prop="c.name">
          </el-table-column>

          <el-table-column
            label="库存"
            width="180">

            <template slot-scope="scope">
              <el-input v-model="scope.row.inp"/>
            </template>

          </el-table-column>
          <el-table-column
            label="价格"
            width="180">
            <template slot-scope="scope">
              <el-input v-model="scope.row.inpp"/>
            </template>
          </el-table-column>
        </el-table>


        <el-form-item v-if="attData.length>0" label="商品参数" prop="name">

          <el-form-item v-for="a in  attData" :key="a.id" :label="a.nameCH">

            <!--  0 下拉框     1 单选框      2  复选框   3  输入框  -->
            <el-input v-if="a.type==3"  v-model="a.ckValues" >

            </el-input>

            <el-input v-if="a.type==3" v-model="a.ckValues"/>

            <el-select v-if="a.type==0" v-model="a.ckValues" placeholder="请选择">
              <el-option v-for="b in a.values" :key="b.id" :label="b.nameCH" :value="b.nameCH"></el-option>
            </el-select>

            <el-radio-group v-model="a.ckValues" v-if="a.type==1">
              <el-radio v-for="b in a.values" :key="b.id" :label="b.nameCH"></el-radio>
            </el-radio-group>

            <el-checkbox-group v-model="a.ckValues" v-if="a.type==2">
              <el-checkbox v-for="b in a.values" :key="b.id" :label="b.nameCH" name="type"></el-checkbox>
            </el-checkbox-group>


          </el-form-item>
          <el-button type="primary" plain @click="comintAdd">提交</el-button>
        </el-form-item>

      </el-form>

    </el-dialog>


  </div>
</template>

<script>
  export default {
    name: "Shop",
    data() {
      return {
        //条件查询的数据
        queryForm: {
          name: ""
        },
        //表格展示的数据
        TableData: [],
        //类型数据
        types: [],
        TypeDatas: [],
        typeName: "",
        productForm2: {},
        attData: [],   //非sku的属性数据
        SKUData: [], //sku属性数据
        //品牌数据
        brandData: [],
        skuTableData: [],
        cols: [],
        //修改表格数据的模板  默认关闭 false
        updateTableData: false,
        //sku修改的模板
        SkuDataTable: false,
        tableShow: false,
        //修改商品表格的表单数据
        updateData: {
          name: "",
          title: "",
          productdecs: "",
          price: 0,
          stocks: 0,
          sortNum: 0,
          imgPath: "",
          bandId: "",
          imgPath: 0,
          typeId: 0,
          id: 0
        },
        //分页的参数
        count: 0,
        sizes: [2, 5, 10, 20],
        size: 2,

      }
    },
    created: function () {
      //查询我们表格展示的数据
      this.queryTableData(1)
      //页面初始化的时候去查询我们的分类数据
      this.queryTypeData();
      //在页面初始化时查询我们的品牌数据
      this.queryBrandData();
    },
    methods: {
      //查询我们的表格数据展示
      queryTableData: function (page) {
        this.$ajax.post("http://localhost:8080/ShopController/queryDataLimit?limit=" + this.size + "&page=" + page + "&" + this.$qs.stringify(this.queryForm)).then(res => {
          this.TableData = res.data.data.list;
          this.count = res.data.data.count;
        }).catch(err => console.log(err));
      }, handleCurrentChange: function (page) { //跳转页面
        this.queryTableData(page);
      }, handleSizeChange: function (size) { //跳转页面
        this.size = size;
        this.queryTableData(1);
      },
      //查询我们的分类数据
      queryTypeData: function () {
        this.$ajax.get("http://localhost:8080/GoodsController/getData").then(res => {
          this.types = res.data.data;  // 把请求的数据  赋给全局
          this.getTypeDatas()
          //遍历所有的子节点
          for (var i = 0; i < this.TypeDatas.length; i++) {
            //定义的全局属性 用来存储我们拼接的底层信息
            this.typeName = "";
            //处理子节点的name属性
            this.chandleName(this.TypeDatas[i]);
            this.TypeDatas[i].name = this.typeName.split("/").reverse().join("/").substring(0, this.typeName.length - 1);
          }
        }).catch(err => console.log(err));
      },
      //循环在表格展示我们的
      changetypeId: function (row) {
        for (let i = 0; i < this.types.length; i++) {
          if (row.typeId == this.types[i].id) {
            return this.types[i].name;
          }
        }
      }
      ,
      queryBrandData: function () {
        this.$ajax.post("http://localhost:8080/BrandController/queryDataAll").then(res => {
          this.brandData = res.data.data;  // 把请求的数据  赋给全局
        }).catch(err => console.log(err));
      }
      ,
      //循环拼接我们的品牌信息
      SpellBrand: function (row) {
        for (let i = 0; i < this.brandData.length; i++) {
          if (row.bandId == this.brandData[i].id) {
            return this.brandData[i].name
          }
        }
      },
      //修改商品表格数据的方法
      toUpdateTable: function (id) {
        //单行查询我们的数据  回显到修改的弹框中
        this.$ajax.post("http://localhost:8080/ShopController/queryDataById?id=" + id).then(res => {
          this.updateData = res.data.data;  // 把请求的数据  赋给全局
          console.log(res)
        }).catch(err => console.log(err));
        this.updateTableData = true;
      },
      uploadImage: function (response, file, fileList) {
        this.updateData.imgPath = response.data;
      },
      updateForm: function () {
        console.log(this.updateData)
        //修改的提交
        this.$ajax.post("http://localhost:8080/ShopController/updataData", this.$qs.stringify(this.updateData)).then(res => {
          //关闭弹框
          this.updateTableData = false;
          this.queryTableData(1);
        }).catch(err => console.log(err));
      },
      upSkuTable: function (row) {
        this.SkuDataTable = true;
        //回显分类id
        this.productForm2.typeId = row.typeId;
        //回显属性数据
        this.getAttrData(row.typeId, row.id);
      },
      //这是一个查询我们type表的一个方法
      getAttrData: function (typeId, pid) {
        this.attData = [];
        this.SKUData = [];
        this.$ajax.post("http://localhost:8080/ShopController/queryProductAttrDataByPid?pid=" + pid).then(res => {
          let datas = res.data.data;

          this.$ajax.post("http://localhost:8080/AttributeController/queryDataBuTypeId?typeId=" + typeId).then(res => {
            // 所有的属性数据
            let attrDatas = res.data.data;
            //判断分类是否有数据   更新 参数和规格
            if (attrDatas.length > 0) {
              //初始化  attData      SKUData
              for (let i = 0; i < attrDatas.length; i++) {
                //判断是否为sku属性
                if (attrDatas[i].isSKU == 1) {


                  if (attrDatas[i].type != 3) {

                    if (attrDatas[i].type == 2) {
                      if (this.getValeu(attrDatas[i].name, datas) == "") {
                        attrDatas[i].ckValues = [];
                      } else {
                        attrDatas[i].ckValues = this.getValeu(attrDatas[i].name, datas);
                      }
                    } else {
                      attrDatas[i].ckValues = this.getValeu(attrDatas[i].name, datas);
                    }

                    //发起请求 查询此属性对应的选项值
                    this.$ajax.post("http://localhost:8080/AttributeValueController/queryDate?attId=" + attrDatas[i].id).then(res => {
                      console.log(res)
                      attrDatas[i].values = res.data.data;
                      this.attData.push(attrDatas[i]);
                    })
                  } else {
                    attrDatas[i].ckValues = this.getValeu(attrDatas[i].name, datas);
                    this.attData.push(attrDatas[i]);
                  }

                } else {
                  if (attrDatas[i].type != 3) {

                    //回显
                    if (attrDatas[i].type == 2) {
                      if (this.getValeu(attrDatas[i].name, datas) == "") {
                        attrDatas[i].ckValues = [];
                      } else {
                        attrDatas[i].ckValues = this.getValeu(attrDatas[i].name, datas);
                      }
                    } else {
                      attrDatas[i].ckValues = this.getValeu(attrDatas[i].name, datas);
                    }


                    //发起请求 查询此属性对应的选项值
                    this.$ajax.post("http://localhost:8080/AttributeValueController/queryDate?attId=" + attrDatas[i].id).then(res => {
                      attrDatas[i].values = res.data.data;
                      attrDatas[i].ckValues = this.getValeu(attrDatas[i].name, datas);
                      this.SKUData.push(attrDatas[i]);
                    })
                  } else {
                    attrDatas[i].ckValues = [];
                    this.SKUData.push(attrDatas[i]);
                  }

                }
              }
              console.log(this.attData)
            } else {
              this.SKUData = [];
              this.attData = [];
            }

          }).catch(err => console.log(err))
        })
      },
      getTypeDatas() {
        for (var i = 0; i < this.types.length; i++) {
          var nnn = this.types[i];
          //判断所有底层节点 并且赋值到下拉框
          this.isParent(nnn)
        }
        console.log(this.TypeDatas)
      },
      isParent: function (datas) {
        var ccc = true;
        for (var j = 0; j < this.types.length; j++) {
          if (datas.id == this.types[j].pid) {
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
          for (var i = 0; i < this.types.length; i++) {
            if (node.pid == this.types[i].id) {
              this.chandleName(this.types[i]);
              break;
            }
          }
        } else {
          this.typeName += "/" + node.name;
        }
      },//监听sku属性 改变事件
      skuChange: function () {


        //清空动态列头
        this.cols = [];
        this.skuTableData = [];

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
          for (var i = 0; i < iiu.length; i++) {
            let valuesAttr = iiu[i];
            let tableValue = {};
            var lie = iiu[i];
            for (var j = 0; j < lie.length; j++) {
              var key = this.cols[j].name;
              tableValue[key] = valuesAttr[j]
            }
            this.skuTableData.push(tableValue)
          }

        } else {
          this.tableShow = false;
        }

      },
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
      getValeu: function (key, data) {
        let arrValue = [];
        //遍历当前商品对应的所有的属性
        for (let i = 0; i < data.length; i++) {
          //得到当前属性数据的一个 将字符串转为json
          let objData = JSON.parse(data[i].attrData);
          // 判断当前的属性数据 是否为想要的属性值
          if (objData[key] != null) { //找到对应的值了
            //判断当前属性是否为sku
            if (objData.inpp != null) {
              if (arrValue.indexOf(objData[key]) == -1) {
                arrValue.push(objData[key]);
              }
            } else {
              return objData[key];
            }

          }
        }
        return arrValue;
      },
      comintAdd: function () {//新增商品
        console.log(this.productForm2.typeId)
        console.log(this.attData)
        console.log(this.skuTableData)
        console.log(this.updateData)
        //声明后台接参的atrr
        this.updateData.typeId = this.productForm2.typeId;
        var atrrs = [];
        //循环数据  吧数据用json串的形式  放到数组中
        for (let i = 0; i < this.attData.length; i++) {
          let attData = {};
          attData[this.attData[i].name] = this.attData[i].ckValues
          atrrs.push(attData);
        }

        this.updateData.attr = JSON.stringify(atrrs);
        this.updateData.sku = JSON.stringify(this.skuTableData); //传参是string   怎么将js json 转为字符串
        console.log(this.$qs.stringify(this.updateData));

        this.$ajax.post("http://localhost:8080/ShopController/addDataSku", this.$qs.stringify(this.updateData)).then(res => {
          this.$message.success("添加成功");
        })
      }




      /*methods结束*/
    }
  }
</script>

<style scoped>

</style>
