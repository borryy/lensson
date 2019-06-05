<template>
    <div class="login">
      <img class="logo" src="../assets/logo.png" alt="">
      <mt-field label="用户名" v-model="userPhone" placeholder="请输入用户名"></mt-field>
      <mt-field label="密码" v-model="userPwd" type="password" placeholder="密码"></mt-field>
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
      userPhone: "",
      userPwd:""
    }
  },
  components:{
    mtField:Field,
    mtButton:Button
  },
  methods:{
    goIndex(){
        let postData = this.$qs.stringify({
          userPwd:this.userPwd,
          userPhone:this.userPhone
        });
        this.$axios({
          method: 'post',
          url: '/course/user/userLogin',
          data:postData,
          }).then(function(response){
            if(response.data.success){
                Toast({
                    message: '登录成功',
                    position: 'bottom',
                    duration: 3000
                });
                this.$router.push({
                    name: 'Home',
                    params: {
                      msg:'success'
                    }
                })
            }
          }.bind(this)).catch(function(error){
            console.log(error);
          });
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
  body{
    background-color: #fff;
  }
</style>
