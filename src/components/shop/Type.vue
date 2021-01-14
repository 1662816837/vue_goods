<template>
  <div>
    <h1>商品分类管理</h1>
    <el-tree
      :data="data"
      show-checkbox
      :props="defaultProps"
      accordion

    >
<span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            @click="() => append(data)">
            新增
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            修改
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => del(node, data)">
            删除
          </el-button>
        </span>
      </span>
    </el-tree>


    <!--  新增的弹框   -->
    <el-dialog title="录入分类产品信息" :visible.sync="addFormFlag" width="500px">

      <el-form :model="addCarForm" ref="addCarForm"   label-width="80px">

        <el-form-item label="名称" prop="name">
          <el-input v-model="addCarForm.name" autocomplete="off" ></el-input>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addFormFlag = false">取 消</el-button>
        <el-button type="primary" @click="addForm()">确 定</el-button>
      </div>
    </el-dialog>

    <!--  修改的弹框   -->
    <el-dialog title="录入分类产品信息" :visible.sync="updateFormFlag" width="500px">

      <el-form :model="updateForm" ref="updateForm"   label-width="80px">

        <el-form-item label="名称" prop="name">
          <el-input v-model="updateForm.name" autocomplete="off" ></el-input>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="updateFormFlag = false">取 消</el-button>
        <el-button type="primary" @click="updateForm2()">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
  export default {
    name: "Type",
    data() {
      return {
        data: [],//tree 的数据
        ajaxData: [],// 远程请求的数据
        addFormFlag:false,
        updateFormFlag:false,
        jsonStr: "", //递归拼接处理
        defaultProps:{},
        addCarForm:{
          name:"",
          pid:"",
        },
        updateForm:{
          name:"",
          id:""
        },
        deleteForm:{
          id:"",
          isDel:""
        }
      }
    },created: function () {
      this.createDate();
    }, methods: {
      chandleData: function () {//ajaxData  处理成 data   //转换数据
        //先找到顶层节点的数据
        for (var i = 0; i < this.ajaxData.length; i++) {
          if (this.ajaxData[i].pid == 0) {
            this.diguiNode(this.ajaxData[i]);
            break;
          }
        }
        //将字符串 转为json对象
        this.data.push(JSON.parse(this.jsonStr));
      },
      diguiNode: function (node) {
        //判断此节点是否为父节点
        var bf = this.isParent(node);
        if(bf==true){
          //{"id":1,"label":"分类",}
          //{"id":1,"label":"分类","children":[]}
          this.jsonStr+='{"id":'+node.id+',"label":"'+node.name+'","children":[';
          //拼接子节点
          //查找此节点的子节点
          let count=0
          for (let i = 0; i <this.ajaxData.length ; i++) {
            //判断是否为当前节点的子节点
            if(node.id==this.ajaxData[i].pid){
              count++;
              this.diguiNode(this.ajaxData[i]);
              this.jsonStr+=",";
            }
          }
          //处理多余的,号
          if(count!=0){
            this.jsonStr=this.jsonStr.substr(0,this.jsonStr.length-1);
          }

          //拼完整
          this.jsonStr+=']}';
        }else{
          this.jsonStr+='{"id":'+node.id+',"label":"'+node.name+'"}';
        }
      },
      isParent: function (node) {
        for (var i = 0; i < this.ajaxData.length; i++) {
          if (node.id == this.ajaxData[i].pid) {
            return true;
          }
        }
        return false;
      },
      append:function (data) {
        this.addFormFlag=true;
        this.addCarForm.pid=data.id;
      },
      addForm:function () {
        //提交表单
        this.$ajax.post("http://localhost:8080/GoodsController/add",this.$qs.stringify(this.addCarForm)).then(res=>{
          //关闭弹框
          this.addFormFlag=false;
          location.reload();
        }).catch(err=>console.log(err));
      },
      createDate:function () {
        //远程请求数据
        this.$ajax.get("http://localhost:8080/GoodsController/getData").then(res => {
          this.ajaxData = res.data.data;  // 把请求的数据  赋给全局
          this.chandleData();
        }).catch(err => console.log(err));
      },
      remove:function (node,data) {
        this.updateForm.name=data.label;
        this.updateForm.id=data.id;
        this.updateFormFlag=true;

        console.log(data)
      },
      updateForm2:function () {
        this.$ajax.post("http://localhost:8080/GoodsController/update",this.$qs.stringify(this.updateForm)).then(res => {
          //关闭弹框
          this.updateFormFlag=false;
          location.reload();
        }).catch(err => console.log(err));
      },
      del:function (node,data) {
        this.deleteForm.id=data.id;
        this.deleteForm.isDel=1;
        this.$ajax.post("http://localhost:8080/GoodsController/update",this.$qs.stringify(this.deleteForm)).then(res => {
          location.reload();
        }).catch(err => console.log(err));
      }


    }
  }
</script>

<style>
  .custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
  }
</style>
