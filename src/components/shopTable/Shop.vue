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
        //品牌数据
        brandData: [],
        //修改表格数据的模板  默认关闭 false
        updateTableData: false,
        //sku修改的模板
        SkuDataTable: false,
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
      },
      //这是一个查询我们type表的一个方法
      getAttrData: function () {

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
      }



      /*methods结束*/
    }
  }
</script>

<style scoped>

</style>
