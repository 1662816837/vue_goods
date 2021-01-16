<template>
  <div>
    <h1>属性管理</h1>

    <div id="searchDiv">
      <el-form :inline="true" :model="searchForm" class="demo-form-inline">

        <el-form-item label="名称">
          <el-input v-model="searchForm.name" placeholder="名称"></el-input>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="queryData(1)">查询</el-button>
          <el-button type="success" @click="addFormFlag=true">新增</el-button>
        </el-form-item>
      </el-form>
    </div>
    <!--表格展示----------------------------------------------------------------------------->
    <div id="attributeTable">
      <el-table
        :data="attributeData"
        style="width: 100%">

        <el-table-column
          prop="id"
          label="序号"
          width="180">
        </el-table-column>

        <el-table-column
          prop="name"
          label="属性名称"
          width="180">
        </el-table-column>

        <el-table-column
          prop="nameCH"
          label="属性中文名"
          width="180">
        </el-table-column>

        <el-table-column
          prop="typeId"
          label="类型"
          :formatter="changetypeId"
        >
        </el-table-column>

        <el-table-column
          prop="type"
          label="展示方式"
          :formatter="changetype"
        >

        </el-table-column>

        <el-table-column
          prop="isSKU"
          label="是否为Sku属性"
          :formatter="isSku"
        >
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
          label="操作">
          <template slot-scope="scope">
            <el-button type="primary" @click="toUpdate(scope.row)">修改</el-button>
            <!--<el-button type="primary" @click="deleteBrand(scope.row.id)">删除</el-button>-->
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
    <!-----------------------新增模板---------------------------------------->
    <!--新增模板-->
    <el-dialog title="品牌信息" :visible.sync="addFormFlag">

      <el-form :model="addForm" ref="addForm" label-width="100px">

        <el-form-item label="属性英文名" prop="name">
          <el-input v-model="addForm.name" autocomplete="off"></el-input>
        </el-form-item>

        <el-form-item label="属性中文名" prop="nameCH">
          <el-input v-model="addForm.nameCH" autocomplete="off"></el-input>
        </el-form-item>


        <el-form-item label="商品类型" prop="typeId">
          <el-select v-model="addForm.typeId" placeholder="请选择">
            <el-option
              v-for="item in TypeDatas"
              :key="item.id"
              :label="item.name"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="属性类型" prop="type">
          <el-radio v-model="addForm.type" :label="0">下拉框</el-radio>
          <el-radio v-model="addForm.type" :label="1">单选框</el-radio>
          <el-radio v-model="addForm.type" :label="2">复选框</el-radio>
          <el-radio v-model="addForm.type" :label="3">输入框</el-radio>
        </el-form-item>

        <el-form-item label="是否为SKU" prop="isSKU">
          <el-radio v-model="addForm.isSKU" :label="0">是</el-radio>
          <el-radio v-model="addForm.isSKU" :label="1">否</el-radio>
        </el-form-item>


      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addFormFlag = false">取 消</el-button>
        <el-button type="primary" @click="addAttributeData">确 定</el-button>
      </div>
    </el-dialog>


  </div>
</template>

<script>
  export default {
    name: "Attribute",
    data() {
      return {
        attributeData: [],
        count: 0,
        sizes: [2, 5, 10, 20],
        size: 2,
        addFormFlag: false,
        TypeDatas: [],
        TypeData: [],
        searchForm: {
          name: ""
        },
        addForm: {
          id: "",
          name: "",
          nameCH: "",
          typeId: "",
          type: "",
          isSKU: "",
        }
      }
    }, created: function () {
      //请求数据
      this.getTypeData();
      this.queryData(1);


    },
    methods: {
      queryData: function (page) {
        //参数格式化
        var searchStr = this.$qs.stringify(this.searchForm);//条件查询的参数
        // console.log(searchStr);
        var url = "http://localhost:8080/AttributeController/queryData?limit=" + this.size + "&page=" + page + "&" + searchStr;
        //发起请求
        this.$ajax.post(url).then(res => {
          this.attributeData = res.data.data.list;
          this.count = res.data.data.count;
        }).catch(err => console.log(err));
      },
      handleCurrentChange: function (page) { //跳转页面
        this.queryData(page);
      },
      handleSizeChange: function (size) { //跳转页面
        this.size = size;
        this.queryData(1);
      },
      changetypeId: function (row, column) {
        for (let i = 0; i < this.TypeData.length; i++) {
          if (row.typeId == this.TypeData[i].id) {
            return this.TypeData[i].name
          }
        }
        return "未知"
      }, getTypeData: function () {
        this.$ajax.get("http://localhost:8080/GoodsController/getData").then(res => {
          console.log(res)
          this.TypeData = res.data.data;
          this.getTypeDatas()
        }).catch(err => console.log(err))
      }, changetype: function (row, column) {
        return row.type == 0 ? "下拉框" : row.type == 1 ? "单选框" : row.type == 2 ? "复选框" : "输入框"
      }, isSku: function (row, column) {
        return row.type == 0 ? "是" : "否"
      }, getTypeDatas() {
        for (var i = 0; i < this.TypeData.length; i++) {
          var rs  =this.isParent(this.TypeData[i])
          if (rs == false) {
            this.TypeDatas.push(this.TypeData[i])
          }
        }
        console.log(this.TypeDatas)

      },
      addAttributeData:function () {
        if (this.addForm==null){
          this.$ajax.post("http://localhost:8080/AttributeController/addData",this.$qs.stringify(this.addForm)).then(res=>{
            //关闭弹框
            this.addFormFlag=false;
            this.queryData(1);
          }).catch(err=>console.log(err));
        } else{
          var data = {"id":this.addForm.id,"name":this.addForm.name,"isSKU":this.addForm.isSKU,"nameCH":this.addForm.nameCH,"type":this.addForm.type,"typeId":this.addForm.typeId};
          this.$ajax.post("http://localhost:8080/AttributeController/updateData",this.$qs.stringify(data)).then(res=>{
            //关闭弹框
            this.addFormFlag=false;
            this.queryData(1);
          }).catch(err=>console.log(err));
        }

      },
      toUpdate:function (row) {
        this.$ajax.post("http://localhost:8080/AttributeController/queryDataById?id="+row.id).then(res=>{
          this.addForm=res.data.data
          this.queryData(1)
        }).catch(err=>console.log(err))
        this.addFormFlag=true;
      }
      ,
      isParent:function (datas) {
        for (var j = 0; j <this.TypeData.length ; j++) {
          if (datas.id==this.TypeData[j].pid){
            return true
          }
        }
        return false
      }

    }
  }
</script>

<style scoped>

</style>
