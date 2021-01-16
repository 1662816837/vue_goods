<template>
    <div>

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

      <div id="carTable">
        <el-table
          :data="brandData"
          style="width: 100%">

          <el-table-column
            prop="id"
            label="序号"
            width="180">
          </el-table-column>

          <el-table-column
            prop="name"
            label="店铺名称"
            width="180">
          </el-table-column>

          <el-table-column
            prop="bandE"
            label="店铺首字母"
            width="180">
          </el-table-column>

          <el-table-column
            prop="bandDesc"
            label="店铺介绍">
          </el-table-column>


          <el-table-column
            prop="imgpath"
            label="图片">
            <!-- 按文本处理   :formatter="formatImg"    -->
            <!-- 模板处理  html  -->
            <template slot-scope="scope">
              <!-- <img src="192.168.1.43:8080/imgFiless/f86a6cd6-a0e3-47a6-ba03-62a68ff41c99.gif"/>-->
              <img width="50px" :src="scope.row.imgpath"/>
            </template>
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
              <el-button type="primary" @click="updateBrand(scope.row.id)">修改</el-button>
              <el-button type="primary" @click="deleteBrand(scope.row.id)">删除</el-button>
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


      <!--  新增的弹框   -->

      <!-- //carname(姓名,String类型),carType(类型,Integer类型),imgPath(图片路径 String类型),price(价格,double类型),bid(品牌,Integer类型),createDate(时间类型)-->
      <el-dialog title="录入汽车信息" :visible.sync="addFormFlag">

        <el-form :model="addCarForm" ref="addCarForm"   label-width="80px">

          <el-form-item label="名称" prop="name">
            <el-input v-model="addCarForm.name" autocomplete="off" ></el-input>
          </el-form-item>

          <el-form-item label="店铺首字母" prop="bandE">
            <el-input v-model="addCarForm.bandE" autocomplete="off" ></el-input>
          </el-form-item>

          <el-form-item label="店铺介绍" prop="bandDesc">
            <el-input v-model="addCarForm.bandDesc" autocomplete="off" ></el-input>
          </el-form-item>
          <el-form-item label="排序" prop="ord">
            <el-input v-model="addCarForm.ord" autocomplete="off" ></el-input>
          </el-form-item>



          <el-form-item label="图片">
            <el-upload
              class="upload-demo"
              action="http://localhost:8080/BrandController/uploadFile"
              :on-success="imgCallBack"
              name="file"
              list-type="picture">
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
          </el-form-item>

        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="addFormFlag = false">取 消</el-button>
          <el-button type="primary" @click="addForm">确 定</el-button>
        </div>
      </el-dialog>


      <!--  修改的弹框   -->

      <!-- //carname(姓名,String类型),carType(类型,Integer类型),imgPath(图片路径 String类型),price(价格,double类型),bid(品牌,Integer类型),createDate(时间类型)-->
      <el-dialog title="录入汽车信息" :visible.sync="updateFormFlag">

        <el-form :model="updateBrandForm" ref="updateBrandForm"   label-width="80px">

          <el-form-item label="名称" prop="name">
            <el-input v-model="updateBrandForm.id" v-if="false" autocomplete="off" ></el-input>
            <el-input v-model="updateBrandForm.name" autocomplete="off" ></el-input>
          </el-form-item>

          <el-form-item label="店铺首字母" prop="bandE">
            <el-input v-model="updateBrandForm.bandE" autocomplete="off" ></el-input>
          </el-form-item>

          <el-form-item label="店铺介绍" prop="bandDesc">
            <el-input v-model="updateBrandForm.bandDesc" autocomplete="off" ></el-input>
          </el-form-item>
          <el-form-item label="排序" prop="ord">
            <el-input v-model="updateBrandForm.ord" autocomplete="off" ></el-input>
          </el-form-item>



          <el-form-item label="图片">
            <img  width="50px" :src="updateBrandForm.imgpath"/>
            <el-upload
              class="upload-demo"
              action="http://localhost:8080/BrandController/uploadFile"
              :on-success="imgCallBack"
              name="file"
              list-type="picture">
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
          </el-form-item>

        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="updateFormFlag = false">取 消</el-button>
          <el-button type="primary" @click="updateForm">确 定</el-button>
        </div>
      </el-dialog>


    </div>
</template>

<script>
    export default {
        name: "Brand",
      data(){
          return{
            brandData:[],
            count: 0,
            sizes:[2,5,10,20],
            size: 2,
            searchForm:{
              name:""
            },
            addFormFlag:false,
            updateFormFlag:false,
            addCarForm:{
              name:"",
              bandE:"",
              bandDesc:"",
              imgpath:"",
              ord:""
            },
            updateBrandForm:{
              id:"",
              name:"",
              bandE:"",
              bandDesc:"",
              imgpath:"",
              ord:""
            }
          }
      },
      created: function () {
        //请求数据
        this.queryData(1);
        //查询品牌数据

      },
      methods:{
        queryData: function (page) {
          //参数格式化
           var searchStr=this.$qs.stringify(this.searchForm);//条件查询的参数
          // console.log(searchStr);
          var url = "http://localhost:8080/BrandController/getData?limit=" + this.size + "&page=" + page + "&"+searchStr;
          console.log(url);
          //发起请求
          this.$ajax.post(url).then(res => {
            console.log(res.data.data.list)
             this.brandData = res.data.data.list;
             this.count = res.data.data.count;
          }).catch(err => console.log(err));
        },
        handleCurrentChange:function(page){ //跳转页面
          this.queryData(page);
        },handleSizeChange:function(size){ //跳转页面
          this.size=size;
          this.queryData(1);
        },
        imgCallBack:function(response, file, fileList){ //图片上传的回调函数
          // 赋值
         this.addCarForm.imgpath=response.data.filePath;
         this.updateBrandForm.imgpath=response.data.filePath;
        },
        addForm:function () {
          this.$ajax.post("http://localhost:8080/BrandController/addData",this.$qs.stringify(this.addCarForm)).then(res=>{
            //关闭弹框
            this.addFormFlag=false;
            this.queryData(1);
          }).catch(err=>console.log(err));
        },
        updateBrand:function (id) {
          this.$ajax.post("http://localhost:8080/BrandController/queryDataById?id="+id+"").then(res=>{

            debugger;
            this.updateBrandForm.id=res.data.data.id;
            this.updateBrandForm.name=res.data.data.name;
            this.updateBrandForm.bandE=res.data.data.bandE;
            this.updateBrandForm.bandDesc=res.data.data.bandDesc;
            this.updateBrandForm.imgpath=res.data.data.imgpath;
            this.updateBrandForm.ord=res.data.data.ord;

            console.log(this.updateCarForm)
            this.updateFormFlag=true;
          }).catch(err=>console.log(err));
          this.updateFormFlag=true;

        },
        updateForm:function () {
          //提交表单
          this.$ajax.post("http://localhost:8080/BrandController/updateDate",this.$qs.stringify(this.updateBrandForm)).then(res=>{
            //关闭弹框
            this.updateFormFlag=false;
            this.queryData(1);
          }).catch(err=>console.log(err));
        },
        deleteBrand:function (id) {
          this.$ajax.post("http://localhost:8080/BrandController/deleteDate?id="+id+"").then(res=>{
            //重新查询数据
            this.queryData(1);
          }).catch(err=>console.log(err));
        }
        
        
      }
    }
</script>

<style scoped>

</style>
