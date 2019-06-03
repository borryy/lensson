<template>
    <div class="login">
      <img class="logo" src="../assets/logo.png" alt="">
      <mt-field label="用户名" placeholder="请输入用户名"></mt-field>
      <mt-field label="密码" type="password" placeholder="密码"></mt-field>
      <mt-button type="primary" @click="goIndex" size="large">登录</mt-button>
    </div>
</template>

<script>
import { Field } from 'mint-ui'
import { Button } from 'mint-ui'
import { Toast } from 'mint-ui'
export default {
  name: "Login",
  data() {
    return {
      msg: "Welcome to Your Vue.js App"
    }
  },
  components:{
    mtField:Field,
    mtButton:Button
  },
  methods:{
    goIndex(){
        Toast({
          message: '登录成功',
          position: 'bottom',
          duration: 5000
        });
       this.$router.push({
          name: 'Index',
          params: {
           
          }
        })
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .login{
    padding: 10px;
    height: 100%;
    box-sizing: border-box;
    text-align: center;
  }
  .login .logo{
    width: 70%;
    margin: 20px auto;
  }
  .login .mint-button--large{
    margin-top: 15px;
    margin-bottom: 15px;
  }
  .login .mint-cell-wrapper{
    background: none!important;
  }
</style>
