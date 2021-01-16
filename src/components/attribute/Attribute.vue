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
          <!--<el-button type="success" @click="addFormFlag=true">新增</el-button>-->
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
          label="是否为Sku属性">
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
            <!--<el-button type="primary" @click="updateBrand(scope.row.id)">修改</el-button>-->
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




  </div>
</template>

<script>
    export default {
        name: "Attribute",
      data(){
          return{
            attributeData:[],
            count: 0,
            sizes:[2,5,10,20],
            size: 2,
            TypeDatas:[],
            TypeData:[],
            searchForm:{
              name:""
            }
          }
      },created: function () {
        //请求数据
        this.queryData(1);
        this.getTypeData();

      },
      methods:{
        queryData: function (page) {
          //参数格式化
          var searchStr=this.$qs.stringify(this.searchForm);//条件查询的参数
          // console.log(searchStr);
          var url = "http://localhost:8080/AttributeController/queryData?limit=" + this.size + "&page=" + page + "&"+searchStr;
          console.log(url);
          //发起请求
          this.$ajax.post(url).then(res => {
             console.log(res.data.data.list)
             this.attributeData = res.data.data.list;
             this.count = res.data.data.count;
          }).catch(err => console.log(err));
        },
        handleCurrentChange:function(page){ //跳转页面
          this.queryData(page);
        },
        handleSizeChange:function(size){ //跳转页面
          this.size=size;
          this.queryData(1);
        },
        changetypeId:function (row, column) {
          for (let i = 0; i <this.TypeData.length ; i++) {
            if (row.typeId==this.TypeData[i].id){
              return this.TypeData[i].name
            }
          }
          return "未知"
        },getTypeData:function () {
          this.$ajax.get("http://localhost:8080/GoodsController/getData").then(res=>{
            //console.log(res)
            this.TypeData=res.data.data;
            this.getTypeDatas()
          }).catch(err=>console.log(err))
        },changetype:function (row, column) {
          return row.type==0?"下拉框":row.type==1?"单选框":row.type==2?"复选框":"输入框"
        },getTypeDatas(){
          for (var i = 0; i <this.TypeData.length ; i++) {
            var rs  =this.isParent(this.TypeData[i])
            if (rs==false){
              this.TypeDatas.push(this.TypeData[i])
              //console.log(this.TypeDatas)
            }
          }
        }
      }
    }
</script>

<style scoped>

</style>
