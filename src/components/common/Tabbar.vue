<template>
  <!-- tabbar导航栏 -->
  <div class="tabbar flex-row flex-jc">
    <div class="tabbar-logo flex-row flex-ac flex-jc">
      <img class="tabbar-logo-img" src="/@/assets/logo.png" alt="logo">
    </div>
    <div class="tabbar-list flex-row flex-ac flex-jc">
      <div class="tabbar-list-item" v-for="(item, index) in navbarList" :key="index" @click="tabbarClick(index)">{{ item.name }}</div>
    </div>
    <div class="tabbar-search flex-row flex-ac flex-jc">
      <el-input class="tabbar-search-input" v-model="input" placeholder="请输入内容"></el-input>
      <el-button class="tabbar-search-button" icon="el-icon-search"></el-button>
    </div>
    <div class="tabbar-login flex-row flex-jc flex-ac" v-if="isLogin">
      <div class="tabbar-login-box flex-row flex-ac flex-jc" @mouseenter="showBigImg" @mouseleave="hiddenBigImg">
        <img @click="toSetting" ref="loginImg" class="tabbar-login-box-img" src="/@/assets/image/profile.png" alt="头像">
      </div>
      <!-- 头像下拉框 -->
      <div ref="loginBigBox" class="tabbar-login-bigbox flex-col flex-ac" @mouseenter="showBigImg" @mouseleave="hiddenBigImg">
        <div class="tabbar-login-bigbox-title">冰-G</div>
        <hr>
        <div class="tabbar-login-bigbox-classify flex-row flex-jsa">
          <div class="tabbar-login-bigbox-classify-item flex-col flex-ac" v-for="(item, index) in classify" :key="index" @click="classifyClick(index)">
            <div class="tabbar-login-bigbox-classify-item-count">{{ item.count }}</div>
            <div class="tabbar-login-bigbox-classify-item-title">{{ item.title }}</div>
          </div>
        </div>
        <hr>
        <div class="tabbar-login-bigbox-options flex-col">
          <div class="tabbar-login-bigbox-options-item flex-row" @click="optionClick(0)">
            <span class="tabbar-login-bigbox-options-item-icon iconfont">&#xe72a;</span>
            <span>个人中心</span>
          </div>
          <div class="tabbar-login-bigbox-options-item flex-row" @click="optionClick(1)">
            <span class="tabbar-login-bigbox-options-item-icon iconfont">&#xe6c8;</span>
            <span>我的收藏</span>
          </div>
        </div>
        <hr>
        <div class="tabbar-login-bigbox-options flex-col">
          <div class="tabbar-login-bigbox-options-item flex-row" @click="logout">
            <span class="tabbar-login-bigbox-options-item-icon iconfont">&#xe64d;</span>
            <span>退出登录</span>
          </div>
        </div>
      </div>
    </div>
    <div class="tabbar-login flex-row flex-ac flex-jc" v-if="!isLogin">
      <a class="tabbar-login-login flex-row flex-ac" @click="toLogin">登录</a>
      <span class="tabbar-login-cut">|</span>
      <a class="tabbar-login-register flex-row flex-ac" @click="toRegister">注册</a>
    </div>
  </div>
  <div class="empty"></div>
  <!-- 登录框 -->
  <el-dialog width="400px" v-model="modelShow" :key="timer">
    <el-form status-icon ref="ruleForm" :model="form">
      <el-row class="dialog-item">
        <!-- <span class="dialog-title" :class="loginType == 1 ? 'login-select': ''" @click="smsLogin">验证码登录</span>
        <span class="dialog-title-cut">|</span> -->
        <span class="dialog-title" :class="loginType == 2 ? 'login-select': ''" @click="pwdLogin">密码登录</span>
      </el-row>
      <div class="dialog-box" v-if="loginType === 1">
        <p class="dialog-item-label">手机号：</p>
        <el-row class="dialog-item">
          <el-input class="search-input" v-model="form.phone" placeholder="+86"></el-input>
        </el-row>
        <!-- 验证码栏块 -->
        <p class="dialog-item-label">验证码：</p>
        <el-row class="dialog-item flex-row flex-jsb">
          <el-input class="dialog-item-code" v-model="form.code" placeholder="请输入验证码"></el-input>
          <el-button v-show="isGetCode" @click="sendCode" class="dialog-item-getcode" type="info">发送验证码</el-button>
          <el-button v-show="!isGetCode" class="dialog-item-getcode" disabled type="info">{{ getCodeTime }}秒后重新发送</el-button>
        </el-row>
      </div>
      <div class="dialog-box" v-if="loginType === 2">
        <p class="dialog-item-label">手机号：</p>
        <el-row class="dialog-item">
          <el-input class="search-input" v-model="form.phone" placeholder="+86"></el-input>
        </el-row>
        <p class="dialog-item-label">密码：</p>
        <el-row class="dialog-item flex-row flex-jsb">
          <el-input class="search-input" type="password" v-model="form.pwd" placeholder="请输入密码" @keyup.enter="login(form)"></el-input>
        </el-row>
        <!-- 验证码栏块 -->
        <!-- <p class="dialog-item-label">验证码：</p>
        <el-row class="dialog-item flex-row flex-jsb">
          <el-input class="dialog-item-code" v-model="form.code" placeholder="请输入验证码"></el-input>
          <img :src="captcha" alt="" class="dialog-item-getcode" @click="changeCaptcha">
        </el-row> -->
      </div>
    </el-form>
    <div class="dialog-footer">
      <el-button class="dialog-register" type="info" @click="login(form)">确 定</el-button>
    </div>
    <!-- qq 微信登录 -->
    <div class="dialog-bottom flex-row flex-jc">
      <div class="dialog-bottom-item flex-row flex-jc">
        <span class="dialog-bottom-item-icon-qq iconfont">&#xe63c;</span>
      </div>
      <div class="dialog-bottom-item flex-row flex-jc">
        <span class="dialog-bottom-item-icon-vx iconfont">&#xe61a;</span>
      </div>
    </div>
  </el-dialog>
  <!-- 注册框 -->
  <el-dialog width="400px" v-model="modelShow2" :key="timer">
    <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-row class="dialog-item">
        <span class="dialog-title2 login-select">注册账号</span>
      </el-row>
      <el-form-item label="手机号：" prop="phone">
        <el-input v-model.number="ruleForm.phone"></el-input>
      </el-form-item>
      <el-form-item label="密码：" prop="pass">
        <el-input type="password" v-model="ruleForm.pass" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="确认密码：" prop="checkPass">
        <el-input type="password" v-model="ruleForm.checkPass" autocomplete="off"></el-input>
      </el-form-item>
      <!-- 验证码栏块 -->
      <!-- <el-form-item label="验证码" prop="code">
        <el-row class="dialog-item flex-row flex-jsb">
          <el-input class="dialog-item-code2" v-model="ruleForm.code" placeholder="请输入验证码"></el-input>
          <img :src="captcha" alt="" class="dialog-item-getcode2" @click="changeCaptcha">
        </el-row>
      </el-form-item> -->
      <el-button class="register-submit" type="info" @click="submitForm('ruleForm')">注册</el-button>
    </el-form>
  </el-dialog>
</template>

<script>
import { register, login, getNavbarList} from '/@/request/api.js'
export default {
  name: 'Tabbar',
  data() {
    var checkAge = (rule, value, callback) => {
      var mobile_mode=/^1[34578]\d{9}$/;
      if (!mobile_mode.test(value)) {
        return callback(new Error('请输入正确的手机号'))
      } else {
        callback()
      }
    }
    var validatePass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入密码'));
      } else {
        if (this.ruleForm.checkPass !== '') {
          this.$refs.ruleForm.validateField('checkPass');
        }
        callback();
      }
    }
    var validatePass2 = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'));
      } else if (value !== this.ruleForm.pass) {
        callback(new Error('两次输入密码不一致!'));
      } else {
        callback();
      }
    }
    return {
      isLogin: false, // 是否登录
      navbarList: [], // 导航栏列表
      input: '',
      // 是否显示登录面板
      modelShow: false,
      // 是否显示注册面板
      modelShow2: false,
      // 登录表单
      form: {
        phone: null,
        code: null,
        pwd: null
      },
      // 验证码时间切换
      isGetCode: true,
      isGetCode2: true,
      getCodeTime: 60,
      getCodeTime2: 60,
      timer: null,
      timer2: null,
      loginType: 2, // 1————验证码登录 2————密码登录
      // 注册表单+验证
      ruleForm: {
        pass: '',
        checkPass: '',
        phone: '',
        code: '',
        captcha: '', // 验证码域名
        timer: null // el-dialog缓存机制搞不了验证码刷新 绑定key可刷新
      },
      rules: {
        pass: [
          { validator: validatePass, trigger: 'blur' }
        ],
        checkPass: [
          { validator: validatePass2, trigger: 'blur' }
        ],
        phone: [
          { validator: checkAge, trigger: 'blur' }
        ]
      },
      classify: [
        {
          id: 0,
          title: '文章',
          count: 0
        },
        {
          id: 1,
          title: '关注',
          count: 0
        },
        {
          id: 2,
          title: '粉丝',
          count: 0
        }
      ], // 头像信息框标题分类
    }
  },
  created() {
    // 判断是否有token
    if (!!this.$store.state.token) {
      this.isLogin = true
    }
    getNavbarList().then(res => {
      if (res.code === 0) {
        this.navbarList = res.data
      }
    }).catch(err => {
      console.log(err)
    })
  },
  mounted() {
    // 获取验证码图片
    this.getCaptcha()
  },
  methods: {
    // 登录
    toLogin() {
      this.modelShow = true
    },
    // 选择登录方式————验证码登录
    smsLogin() {
      this.loginType = 1
    },
    // 选择登录方式————密码登录
    pwdLogin() {
      this.loginType = 2
    },
    // 注册
    register() {
      console.log('register');
    },
    // 栏目列表点击
    tabbarClick(index) {
      console.log(index)
    },
    // 登录
    login(form) {
      if (this.loginType === 1) {
        // 验证码登录
        var mobile_mode=/^1[34578]\d{9}$/;
        if (!mobile_mode.test(this.form.phone)) {
          this.$message({
            type: 'info',
            message: '请输入正确的手机号'
          })
        } else {
          delete this.form.pwd
          console.log(form)
          this.modelShow = false
        }
      } else {
        // 手机号密码登录
        var mobile_mode=/^1[34578]\d{9}$/;
        if (!mobile_mode.test(this.form.phone)) {
          this.$message({
            type: 'info',
            message: '请输入正确的手机号'
          })
        } else {
          delete this.form.code
          const data = {
            phone: form.phone,
            password: form.pwd
          }
          login(data).then(res => {
            if(res.code === 2) {
              this.$message({
                type: "success",
                message: res.msg
              })
              // 登录成功保存token到localstorage跟vuex中
              const token = JSON.stringify(res.token)
              localStorage.setItem('token', token)
              this.$store.commit('setToken')
              this.modelShow = false
              this.isLogin = true
            }
          }).catch(err => {
            this.$message({
                type: "info",
                message: err.msg
              })
          })
        }
      }
    },
    // 获取验证码图片
    getCaptcha() {
      const num = Math.ceil(Math.random() * 100)
      this.captcha = 'https://xchoy.com/api.php/login/verify?' + num
    },
    // 变换验证码
    changeCaptcha() {
      const num = Math.ceil(Math.random() * 100)
      this.captcha = 'https://xchoy.com/api.php/login/verify?' + num
      this.timer = new Date()
    },
    // 发送登录验证码
    sendCode() {
      var mobile_mode=/^1[34578]\d{9}$/;
      if (!mobile_mode.test(this.form.phone)) {
        this.$message({
          type: 'info',
          message: '请输入正确的手机号'
        })
      } else {
        // 60秒内不允许再发送验证码
        const TIME_COUNT = 60;
        if (!this.timer) {
          this.getCodeTime = TIME_COUNT;
          this.isGetCode = false;
          this.timer = setInterval(() => {
            if (this.getCodeTime > 0 && this.getCodeTime <= TIME_COUNT) {
              this.getCodeTime--;
            } else {
              this.isGetCode = true;
              clearInterval(this.timer);
              this.timer = null;
            }
          }, 1000)
        }
      }
    },
    // 发送注册验证码
    sendCode2() {
      var mobile_mode=/^1[34578]\d{9}$/;
      if (!mobile_mode.test(this.ruleForm.phone)) {
        this.$message({
            type: 'info',
            message: '请输入正确的手机号'
          })
      } else {
        // 60秒内不允许再发送验证码
        const TIME_COUNT 
        = 60;
        if (!this.timer2) {
          this.getCodeTime2 = TIME_COUNT;
          this.isGetCode2 = false;
          this.timer2 = setInterval(() => {
            if (this.getCodeTime2 > 0 && this.getCodeTime2 <= TIME_COUNT) {
              this.getCodeTime2--;
            } else {
              this.isGetCode2 = true;
              clearInterval(this.timer2);
              this.timer2 = null;
            }
          }, 1000)
        }
      }
    },
    // 关闭登录弹窗
    closeForm() {
      this.form = {
        phone: null,
        code: null,
        pwd: null
      }
      this.loginType = 2
    },
    // 注册
    toRegister() {
      this.modelShow2 = true
    },
    // 注册提交
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          let data = {
            phone: this.ruleForm.phone,
            password: this.ruleForm.pass,
            code: this.ruleForm.code
          }
          console.log(data)
          register(data).then(res => {
            console.log(res)
            // 注册成功
            if(res.code === 0){
              this.$message({
                type: 'success',
                message: res.msg
              })
              this.modelShow2 = false
            } else {
              // 注册失败
              this.$message({
                type: 'info',
                message: res.msg
              })
            }
          }).catch(err=> {
            this.$message({
              type: 'info',
              message: err
            })
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    // 聚焦头像显示信息框
    showBigImg() {
      this.$refs.loginBigBox.style.visibility = "visible"
      this.$refs.loginImg.style.width = "68px"
      this.$refs.loginImg.style.height = "68px"
      this.$refs.loginImg.style.top = "15px"
      this.$refs.loginBigBox.style.opacity = "1"
    },
    // 失焦头像隐藏信息框
    hiddenBigImg() {
      this.$refs.loginBigBox.style.visibility = "hidden"
      this.$refs.loginImg.style.width = "40px"
      this.$refs.loginImg.style.height = "40px"
      this.$refs.loginImg.style.top = "0px"
      this.$refs.loginBigBox.style.opacity = "0"
    },
    // 头像下拉框类别跳转
    classifyClick(index) {
      console.log('类别跳转', index)
    },
    // 头像下拉框选项跳转
    optionClick(index) {
      console.log('选项跳转', index)
    },
    // 注销登录
    logout() {
      localStorage.clear()
      this.$store.commit('clearToken')
      this.isLogin = false
      this.$message({
        type: 'success',
        message: '成功退出登录'
      })
    },
    // 点击头像跳转到设置页
    toSetting() {
      this.$router.push({ name: 'setting' })
    }
  }
}

</script>
<style lang="scss" scoped>
  .empty{
    height: 56px;
    width: 100%;
  }
  .tabbar{
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    height: 56px;
    width: 100%;
    background: white;
    box-shadow: 0px 2px 4px rgba(0,0,0,0.1);
    &-logo{
      height: 56px;
      width: 15%;
      &-img{
        width: 100px;
        height: 56px;
      }
    }
    &-list{
      height: 56px;
      width: 45%;
      min-width: 600px;
      &-item{
        padding: 0 16px;
        color: #545C63;
        text-align: center;
        font-size: 16px;
        height: 56px;
        line-height: 56px;
        width: 100px;
        &:hover{
          cursor: pointer;
        }
      }
    }
    &-search{
      height: 56px;
      width: 20%;
      &-input{
        width: 300px;
      }
      &-button{
        width: 50px;
        margin-left: -10px;
      }
    }
    &-login{
      height: 56px;
      width: 20%;
      padding: 0 20px;
      position: relative;
      & a:hover{
        cursor:pointer;
      }
      &-cut{
        color: #bbbbbb;
      }
      &-login{
        margin-right: 10px;
        height: 56px;
        width: 60px;
        &:hover{
          color: rgb(41, 166, 250);
        }
      }
      &-register{
        margin-left: 20px;
        height: 56px;
        width: 60px;
        &:hover{
          color: rgb(41, 166, 250);
        }
      }
      &-box{
        width: 50px;
        height: 50px;
        &-img{
          width: 40px;
          height: 40px;
          position: relative;
          border-radius: 50%;
          cursor: pointer;
          z-index: 9999;
          transition: all 0.15s;
          transition-delay: 0.15s;
        }
      }
      &-bigbox{
        background: white;
        width: 250px;
        height: 290px;
        visibility: hidden;
        opacity: 0;
        border-radius: 5px;
        box-shadow: 0px 0px 5px rgba(0,0,0,0.1);
        position: absolute;
        top: 50px;
        transition: all 0.15s ease 0.15s;
        &-title{
          width: 100%;
          font-size: 16px;
          font-weight: 600;
          color: #585757;
          margin-top: 30px;
          text-align: center;
        }
        hr{
          width: 100%;
          margin: 10px 0;
          border: none;
          height: 1px;
          background-color: #e7e7e7;
        }
        &-classify{
          width:100%;
          &-item{
            width: 25%;
            cursor: pointer;
            &-count{
              font-size: 18px;
              font-weight: 600;
              color: #585757;
            }
            &-title{
              font-size: 14px;
              color: #9e9e9e;
              margin-top: 6px;
            }
          }
        }
        &-options{
          width: 100%;
          &-item{
            width: 100%;
            height: 40px;
            line-height: 40px;
            font-size: 14px;
            color: #555666;
            cursor: pointer;
            &:hover{
              background: #f0f0f5;
            }
            &-icon{
              color: #a2a3af;
              padding: 0 16px 0 30px;
            }
          }
        }
      }
    }
  }
.dialog{
  &-title{
    color: #b3b3b3;
    font-size: 20px;
    font-weight: bold;
    &:hover{
      cursor: pointer;
    }
    &-cut{
      font-size: 20px;
      margin: 0 10px;
    }
  }
  &-title2{
    color: #b3b3b3;
    font-size: 20px;
    font-weight: bold;
    margin-left: 10px;
  }
  &-item{
    &-label{
      margin-bottom: 10px;
    }
  }
  &-bottom{
    height: 50px;
    width: 70%;
    margin: 0 auto;
    &-item{
      width: 50%;
      margin-top: 20px;
      &-icon-qq{
        cursor: pointer;
        font-size: 45px;
        color: #c2c2c2;
        &:hover{
          transition: all 0.5s;
          color: rgb(0, 183, 255);
        }
      }
      &-icon-vx{
        cursor: pointer;
        font-size: 40px;
        color: #c2c2c2;
        &:hover{
          transition: all 0.5s;
          color: rgb(28, 190, 13);
        }
      }
    }
  }
}
// 登录方式选中样式
.login-select{
  color: #606266;
}
// 修改el组件样式
:deep(.tabbar-search-input){
  width: 250px;
  margin-right: 20px;
}
:deep(.dialog-item){
  margin-bottom: 20px;
}
:deep(.dialog-register){
  margin-top: 10px;
  width: 360px;
}
:deep(.dialog-item-code){
  width: 220px;
}
:deep(.dialog-item-getcode){
  width: 130px;
}
:deep(.dialog-item-code2){
  width: 120px;
}
:deep(.dialog-item-getcode2){
  width: 100px;
}
:deep(.register-submit){
  width: 360px;
}
:deep(.el-form-item__label){
  text-align: center;
}

</style>