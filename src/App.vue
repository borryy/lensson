<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>

export default {
  name: 'App',
  data(){
    return{
      testMsg: ''
    }
    
  },
  mounted(){
    this.getItemList()
  },
  methods:{
    getItemList(){
      var that = this
      // that.$axios.get('api/test')
      //   .then(function (response) {
      //     that.testMsg = response.data
      //   })
      //   .catch(function (error) {
      //     console.log(error);
      //   })
    }
  }
}
</script>

<style>
*{
  margin: 0;
  padding: 0;
}
a{
  text-decoration: none;
}
body,html{
  height: 100%;
}
#app {
  height:100%;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
