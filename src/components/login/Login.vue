<template>
  <div class="login-wrap">
    <div class="ms-login">
      <div class="ms-title" style="color: #97a8be">后台管理系统</div>
      <el-form :model="loginForm" :rules="rules" ref="loginForm" label-width="0px" class="ms-content">
        <el-form-item prop="name">
          <el-input v-model="loginForm.name" placeholder="请输入账号">
            <el-button slot="prepend" icon="el-icon-lx-people"></el-button>
          </el-input>
        </el-form-item>
        <el-form-item prop="psw">
          <el-input
            type="password"
            placeholder="请输入密码"
            v-model="loginForm.psw"
            @keyup.enter.native="userLogin"
          >
            <el-button slot="prepend" icon="el-icon-lx-lock"></el-button>
          </el-input>
        </el-form-item>
        <div class="login-btn">
          <el-button type="primary" @click="userLogin">登录</el-button>
          <br/>
          <el-button type="primary" @click="zhuceform">注册</el-button>
        </div>
        <p class="login-tips">Tips : 用户名和密码随便填。</p>
      </el-form>
    </div>

    <!--用户注册弹框-->
    <el-dialog title="注册信息" :visible.sync="showAddForm">
      <el-form :model="zhuCeForm" ref="zhuCeForm" label-width="100px" lef="commodityAddForm"
      >
        <el-form-item label="账号" prop="name">
          <el-input v-model="zhuCeForm.name" width="200px"></el-input>
        </el-form-item>

        <el-form-item label="真实名称" prop="realName">
          <el-input v-model="zhuCeForm.realName" width="200px"></el-input>
        </el-form-item>

        <el-form-item label="密码" prop="password">
          <el-input placeholder="请输入密码" v-model="zhuCeForm.psw" show-password></el-input>
        </el-form-item>

        <el-form-item label="性别" prop="sex">
          <el-radio v-model="zhuCeForm.sex" :label="0" border>女</el-radio>
          <el-radio v-model="zhuCeForm.sex" :label="1" border>男</el-radio>
        </el-form-item>

        <el-form-item label="手机号" prop="phone">
          <el-input v-model.number="zhuCeForm.phone"></el-input>
        </el-form-item>

        <el-form-item label="邮箱" prop="email">
          <el-input v-model="zhuCeForm.email" width="200px"></el-input>
        </el-form-item>

        <el-form-item label="身份证号" prop="idCard">
          <el-input v-model="zhuCeForm.idCard" width="200px"></el-input>
        </el-form-item>

        <el-form-item label="生日" prop="birthday">
          <el-date-picker
            v-model="zhuCeForm.bir"
            type="date"
            placeholder="选择日期">
          </el-date-picker>
        </el-form-item>

        <el-form-item label="头像" prop="imgUrl">
          <el-upload
            class="upload-demo"
            action="http://localhost:8082/api/brand/upload"
            :on-success="imgCallBack"
            name="file"
            list-type="picture">
            <el-button size="small" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>
        </el-form-item>

        <el-form-item label="学历" prop="eduId">
          <el-select v-model="zhuCeForm.eduId" placeholder="请选择">
            <el-option
              v-for="item in eduIdData"
              :key="item.id"
              :label="item.name"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="部门" prop="deptId">
          <el-select v-model="zhuCeForm.deptId" placeholder="请选择">
            <el-option
              v-for="item in deptData"
              :key="item.id"
              :label="item.name"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="success" @click="addUserData">提交</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
    export default {
        name: "Login",
      data: function () {

        return {
          //注册相关的数据
          showAddForm: false,
          zhuCeForm: {
            name: "",
            realName: "",
            psw: "",
            sex: "",
            phone: "",
            email: "",
            idCard: "",
            bir: "",
            imgPath: "",
            eduId: "",
            deptId: ""
          },
          //学历数据
          eduIdData: [{"id": 1, "name": "幼儿园"}, {"id": 2, "name": "小学"}, {"id": 3, "name": "初中"}, {
            "id": 4,
            "name": "幼儿园"
          }, {"id": 5, "name": "大学"}],//学历数据
          //部门相关的数据
          deptData: [{"id": 1, "name": "财务部"}, {"id": 2, "name": "技术部"}, {"id": 3, "name": "销售部"},{"id": 4, "name": "生产部"}],
          loginForm: {
            name: '',
            psw: '',
          },
          rules: {
            name: [{required: true, message: '请输入用户名', trigger: 'blur'}],
            psw: [{required: true, message: '请输入密码', trigger: 'blur'}],
          },
        }
      },
      methods: {

        //登录
        userLogin: function () {
          this.$refs['loginForm'].validate(re => {
            if (re) {
              this.$axios.post("http://localhost:8082/api/login/userLogin",this.$qs.stringify(this.loginForm)).then(res => {
                if (res.data.data.status == 1) {
                  this.$message.success("登录成功");
                  this.$router.push('/home');
                }
                if (res.data.data.status == 0) {
                  this.$message.warning("密码错误");
                }
                if (res.data.data.status == 2) {
                  this.$message.warning("账号不存在");
                }
              }).catch(err => {
                this.$message.error("登录异常");
              })
            }
          })

        },
        //注册弹框
        zhuceform: function () {
          this.showAddForm = true;
        },
        //查询部门数据
       /* queryDeptData: function () {
          this.$axios.get(" http://localhost:8080/api/dept/deptList").then(res => {
            this.deptData = res.data.data;
          }).catch(err => {
            this.$message.error("查询部门异常");
          })
        },*/
        //图片上传的回调函数
        imgCallBack: function (response, file, fileList) {
          this.zhuCeForm.imgPath = response.data;
        },
        //提交注册信息
        addUserData: function () {
          this.$axios.post("http://localhost:8082/api/user/addUser", this.$qs.stringify(this.zhuCeForm)).then(res => {
            if (res.data.code == 200) {
              this.$message.success("注册成功");
              this.showAddForm = false;
            }
            if (res.data.code == 500) {
              this.$message.warning("注册失败");
            }
          }).catch(err => {
            this.$message.error("注册异常");
          });
        }
      },
    }
</script>

<style scoped>
  .login-wrap {
    position: relative;
    width: 100%;
    height: 100%;
    background-image: url(../../assets/img/login-bg.jpg);
    background-size: 100%;
  }

  .ms-title {
    width: 100%;
    line-height: 50px;
    text-align: center;
    font-size: 20px;
    color: #fff;
    border-bottom: 1px solid #ddd;
  }

  .ms-login {
    position: absolute;
    left: 50%;
    top: 50%;
    width: 350px;
    margin: -190px 0 0 -175px;
    border-radius: 5px;
    background: rgba(255, 255, 255, 0.3);
    overflow: hidden;
  }

  .ms-content {
    padding: 30px 30px;
  }

  .login-btn {
    text-align: center;
  }

  .login-btn button {
    width: 100%;
    height: 36px;
    margin-bottom: 10px;
  }

  .login-tips {
    font-size: 12px;
    line-height: 30px;
    color: #fff;
  }
</style>
