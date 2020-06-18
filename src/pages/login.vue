<template>
  <div class="login">
    <div class="container">
      <a href="/#/index"><img src="/imgs/login-logo.png" alt=""></a>
    </div>
    <div class="wrapper">
      <div class="container">
        <div class="login-form" v-if="loginIf">
          <h3>
            <span class="checked">帐号登录</span><span class="sep-line">|</span><span>扫码登录</span>
          </h3>
          <div class="input">
            <input type="text" placeholder="请输入帐号" v-model="username">
          </div>
          <div class="input">
            <input type="password" placeholder="请输入密码" v-model="password">
          </div>
          <div class="btn-box">
            <a href="javascript:;" class="btn" @click="login">登录</a>
          </div>
          <div class="tips">
            <div class="sms">手机短信登录/注册</div>
            <div class="reg" @click="tz">立即注册<span>|</span>忘记密码？</div>
          </div>
        </div>
        <div class="login-form" v-if="!loginIf">
          <h3>
            <span class="checked">小米帐号注册</span>
          </h3>
          <div class="input">
            <input type="text" placeholder="请输入帐号" v-model="username">
          </div>
          <div class="input">
            <input type="password" placeholder="请输入密码" v-model="password">
          </div>
          <div class="input">
            <input type="text" placeholder="请输入邮箱" v-model="email">
          </div>
          <div class="btn-box">
            <a href="javascript:;" class="btn" @click="register">注册</a>
          </div>
          <div class="tips">
            <el-checkbox v-model="checked">已阅读并同意：小米 用户协议和 隐私政策</el-checkbox>
          </div>
        </div>
      </div>
    </div>
    <div class="footer">
      <div class="footer-link">
          <a href="http://yiqi0711.com" target="_blank">张翼麒</a><span>|</span>
          <a href="#" target="_blank">XIAOMI</a><span>|</span>
          <a href="http://www.baidu.com" target="_blank">百度</a><span>|</span>
          <a href="#" target="_blank">SpringBoot仿小米电商项目</a>
      </div>
      <p class="copyright">Copyright ©2019 mi.futurefe.com All Rights Reserved.</p>
    </div>
  </div>
</template>
<script>
import { mapActions } from 'vuex';
export default {
  name: 'login',
  data(){
    return {
      loginIf: true,
      username:'',
      password:'',
      email: '',
      userId:'',
      checked: false
    }
  },
  methods:{
    tz(){
      this.loginIf=false
    },
    login(){
      let { username,password } = this;
      this.axios.post('/user/login',{
        username,
        password
      }).then((res)=>{
        this.$cookie.set('userId',res.id,{expires:'Session'});
        // this.$store.dispatch('saveUserName',res.username);
        this.saveUserName(res.username);
        this.$router.push({
          name:'index',
          params:{
            from:'login'
          }
        });
      }).catch((err) => {
       if (err.status===3)  this.$message.warning('密码不能为空');
       if (err.status===11)  this.$message.warning('用户名或密码错误');
      })
    },
    ...mapActions(['saveUserName']),
    register(){
      if (this.checked!==false) {
        this.axios.post('/user/register',{
          username:this.username,
          password:this.password,
          email:this.email
        }).then(()=>{
          this.$message.success('注册成功');
          this.loginIf=true
        }).catch((err) => {
          if (err.status===2)  this.$message.warning('用户名已存在');
          if (err.status===4)  this.$message.warning('邮箱已存在');
        })
      }else {
        this.$message.warning('请同意小米用户协议');
      }

    }
  }
}
</script>
<style lang="scss">
.login{
  &>.container{
    height:113px;
    img{
      width:auto;
      height:100%;
    }
  }
  .wrapper{
    background:url('/imgs/login-bg.jpg') no-repeat center;
    .container{
      height:576px;
      .login-form{
        box-sizing: border-box;
        padding-left: 31px;
        padding-right: 31px;
        width:410px;
        height:510px;
        background-color:#ffffff;
        position:absolute;
        bottom:29px;
        right:0;
        h3{
          line-height:23px;
          font-size:24px;
          text-align:center;
          margin:40px auto 49px;
          .checked{
            color:#FF6600;
          }
          .sep-line{
            margin:0 32px;
          }
        }
        .input{
          display:inline-block;
          width:348px;
          height:50px;
          border:1px solid #E5E5E5;
          margin-bottom:20px;
          input{
            width: 100%;
            height: 100%;
            border: none;
            padding: 18px;
          }
        }
        .btn{
          width:100%;
          line-height:50px;
          margin-top:10px;
          font-size:16px;
        }
        .tips{
          margin-top:14px;
          display:flex;
          justify-content:space-between;
          font-size:14px;
          cursor:pointer;
          .sms{
            color:#FF6600;
          }
          .reg{
            color:#999999;
            span{
              margin:0 7px;
            }
          }
        }
      }
    }
  }
  .footer{
    height:100px;
    padding-top:60px;
    color:#999999;
    font-size:16px;
    text-align:center;
    .footer-link{
      a{
        color:#999999;
        display:inline-block;
      }
      span{
        margin:0 10px;
      }
    }
    .copyright{
      margin-top:13px;
    }
  }
}
</style>
