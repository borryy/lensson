<template>
  <div class="index">
    <div class="head">
      <img src="../assets/logo.png" alt>
      <h3>{{stuName}}</h3>
    </div>
    <div class="history">
      <mt-button type="primary" size="small" @click="LoadRight()">历史</mt-button>
      <mt-popup v-model="popupVisible" position="right" >
        <div class="rightContent">
          <h4>历史评价</h4>
          <div class="list" v-for="(item,index) in echartsList" :key="index" @click="LoadEc(item.lessonId,item.lessonTime)">
            <mt-cell :title="item.lessonName+'('+ item.lessonTime +')'" is-link></mt-cell>
          </div>
         
        </div>
      </mt-popup>
    </div>
    <div id="myChart"></div>
    <div class="appraise">
      <h3>评价 <small>{{time}}评价</small></h3> 
      <div class="content">
          <h5>研学学分评价</h5>
          <p>{{content}}</p>
      </div>
    </div>
  </div>
</template>

<script>
import { Button } from 'mint-ui'
import { Popup } from 'mint-ui'
import { Cell } from 'mint-ui'
export default {
  name: "Index",
  data() {
    return {
      popupVisible: false,
      echartsJson:[],
      echValue:[],
      echartsList:[],
      content:'',
      time:'',
      stuName:''
    };
  },
  components: {
    mtButton:Button,
    mtPopup:Popup,
    mtCell:Cell
  },
  mounted() {
    console.log(this.$route.params.studentId)
    if(this.$route.params.studentId==undefined){
      this.$router.push({
            name: 'Student',
            params: {
              
            }
        })
    }
    this.showEcharts();
   

  },
  methods: {
    showEcharts() {
      if(this.time == ''){
        this.time='最近一次'
      }
      this.stuName = this.$route.params.stuName
      var ids = this.$route.params.studentId
      let postData = this.$qs.stringify({
          studentId:ids
        });
        this.$axios({
          method: 'post',
          url: '/course/echarts/queryRadar',
          data:postData,
          }).then(function(response){
            if(response.data.success){
                this.echartsJson = response.data.data
                this.loadEchart()
                this.loadpj()
            }
          }.bind(this)).catch(function(error){
            console.log(error);
          });
      
    },
    loadEchart(){
      // 基于准备好的dom，初始化echarts实例
      let myChart = this.$echarts.init(document.getElementById("myChart"));
      // 绘制图表
      let option = {
        title: {
          text: "八项指标分部",
          subtext: this.time
        },
        tooltip: {
          trigger: "axis"
        },
        // legend: {
        //   x: "center",
        //   data: ["小明同学"]
        // },
        toolbox: {
          show: true,
          feature: {
            mark: { show: true },
            // dataView: { show: true, readOnly: false },
            // magicType : {show: true, type: ['line', 'bar', 'stack', 'tiled']},
            // restore: { show: true },
            // saveAsImage: { show: true }
          }
        },
        calculable: true,
        polar: [
          {
            indicator: this.echartsJson,
            radius: 80
          }
        ],
        series: [
          {
            name: "完全实况球员数据",
            type: "radar",
            lineStyle:{
              normal:{
                width:1,
                color:"#ddd"
              }
            },
            itemStyle: {
              normal: {
                areaStyle: {
                  type: "default",
                  color:"#26a2ff",
                   shadowColor: 'rgba(0, 0, 0, 0.5)',
                  shadowBlur: 10,
                  
                }
                
              }
            },
            data: [
              
              {
                value: this.echartsJson.map(function(item){ return item.value}),
                name: "小明同学",
                label: {
                        normal: {
                            show: true,
                            formatter:function(params) {
                                return params.value;
                            },
                            textStyle: {
                            color: '#333'
                        }
                        },

                },
                
              }
            ]
          }
        ]
      };

      myChart.setOption(option);
      window.onresize = function () {
          myChart.resize(); //使第一个图表适应
      }
    },
    loadpj(){
      var ids = this.$route.params.studentId
      let postData = this.$qs.stringify({
          studentId:ids
        });
        this.$axios({
          method: 'post',
          url: '/course/comment/queryCommentByLessonIdOrStudentId',
          data:postData,
          }).then(function(response){
            if(response.data.success){
                this.content = response.data.data.content
            }
          }.bind(this)).catch(function(error){
            console.log(error);
          });
    },
    loadpj1(lessonId){
      var ids = this.$route.params.studentId
      let postData = this.$qs.stringify({
          studentId:ids,
          lessonId:lessonId
        });
        this.$axios({
          method: 'post',
          url: '/course/comment/queryCommentByLessonIdOrStudentId',
          data:postData,
          }).then(function(response){
            if(response.data.success){
                this.content = response.data.data.content
            }
          }.bind(this)).catch(function(error){
            console.log(error);
          });
    },
    LoadEc(lessonId,time){
      this.time = time
      this.loadpj1(lessonId)
      var ids = this.$route.params.studentId
      let postData = this.$qs.stringify({
          studentId:ids,
          lessonId:lessonId
        });
        this.$axios({
          method: 'post',
          url: '/course/echarts/queryRadar',
          data:postData,
          }).then(function(response){
            if(response.data.success){
                this.echartsJson = response.data.data
                this.loadEchart()
                this.popupVisible = !this.popupVisible
            }
          }.bind(this)).catch(function(error){
            console.log(error);
          });
    },
    LoadRight() {
      var ids = this.$route.params.studentId
      this.popupVisible = !this.popupVisible
      let postData = this.$qs.stringify({
          studentId:ids
        });
        this.$axios({
          method: 'post',
          url: '/course/lessonStudent/queryLessonListByStudentId',
          data:postData,
          }).then(function(response){
            if(response.data.success){
                this.echartsList = response.data.data.rows
                
                
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
.head {
  padding: 5px 15px;
  margin: 15px 0;
  box-sizing: border-box;
  display: flex;
  justify-content: center;
  background-color: #eee;
}
.head img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background-color: beige;
}
.head h3 {
  flex-grow: 1;
  background-color: #eee;
  justify-content: center;
  align-items: center;
  display: flex;
}
.appraise {
  padding: 15px;
  box-sizing: border-box;
  font-family: 微软雅黑;
}
.appraise h3{
  text-align: left;
  color: #333;
  
}
.appraise h3 small{
  display: block;
  font-size: 12px;
  color: #aaa;
  font-weight: normal;
  margin-top: 5px;
}
.appraise .content h5{
  text-align: center;
  font-size: 16px;
  margin: 10px 0;
}
.appraise .content {
  padding: 10px;
  font-size: 14px;
  box-sizing: border-box;
  
}
.appraise .content p{
  text-indent: 2em;
}
.history{
  position:absolute;
  top: 90px;
  right: 0;
  padding:10px;
  box-sizing: border-box;
  text-align: right;
  z-index: 10000;
}
.mint-popup-right ,.rightContent{
  width: 100%;
  height: 100%;
  text-align: left;
}
.rightContent h4{
  text-align: center;
  padding: 15px;
}
.list .mint-cell:last-child{
  background: none;
}
.list:last-child{
  border-bottom: 1px solid #d9d9d9;
} 
#myChart {
  width: 100%;
  padding: 10px;
  box-sizing: border-box;
  height: 400px;
}
</style>
