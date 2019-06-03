<template>
  <div class="home">
    <mt-tab-container v-model="selected">
      <mt-tab-container-item id="home">
        <div class="top">
          <h4>研学学分数据统计</h4>
        </div>
        <div class="studentSum item">
          <h3>学生总数</h3>
          <div id="studentSum" class="echart"></div>
        </div>
        <div class="section item">
          <h3>分数区间数量分布</h3>
          <div id="section" class="echart"></div>
        </div>
        <div class="dimension item">
          <h3>各维度满分学生分布</h3>
          <div id="dimension" class="echart"></div>
        </div>
        <div class="progress item">
          <h3>进步最快的学生Top3</h3>
          <div id="progress" class="echart"></div>
        </div>
        <div class="lately item">
          <h3>最近一次各维度评分Top3</h3>
          <div id="lately" class="echart"></div>
        </div>
      </mt-tab-container-item>
      <mt-tab-container-item id="page">
        <div class="top">
          <h4>配置参数</h4>
        </div>
        <div class="configure">
          <mt-field label="进步最快的学生" placeholder="请输入前几名" v-model="progressNum"></mt-field>
          <mt-field label="最近一次各维度评分" placeholder="请输入前几名" v-model="latelyNum"></mt-field>
        </div>
        <div class="save">
          <mt-button type="primary" size="large">保存</mt-button>
        </div>
      </mt-tab-container-item>
      <mt-tab-container-item id="list1">
        <div class="top">
          <h4>学生列表</h4>
        </div>
        <div class="operat">
          <div class="serch">
            <input type="text">
            <mt-button type="primary" size="small">搜索</mt-button>
          </div>
          <mt-button type="primary" size="small" @click="LoadAddStudent()">新增</mt-button>
          <mt-popup v-model="popupStudent" position="right" >
            <div class="rightContent">
              <h4>{{insertStudentList.title}}</h4>
              <input type="hidden" v-model="insertStudentList.id">
              <mt-field label="学生姓名" v-model="insertStudentList.stuName" placeholder="请输入学生姓名" ></mt-field>
              <mt-field label="手机号" v-model="insertStudentList.stuPhone" placeholder="请输入手机号"></mt-field>
              <!-- <mt-field label="添加课程" placeholder="请选择课程">
                 <mt-button type="primary" size="small" @click="LoadClassList()">选择</mt-button>
              </mt-field> -->
              <div class="text-center m-30">
                <mt-button type="primary" size="small" @click="insertStudent()">确定</mt-button>
                <mt-button type="primary" size="small" @click="LoadAddStudent()">返回</mt-button>
              </div>
              
            </div>
          </mt-popup>
          
        </div>
        <div class="studentList" v-if="temp">
          <mt-cell-swipe
          :key="index"  v-for="(n,index) in studentList"
          :right="[
              {
                content: '删除',
                style: { background: '#ff7900', color: '#fff', fontSize:'12px', padding:'0 15px'},
                handler: () => deleteSection(index)
              },
              {
                content: '编辑',
                style: { background: '#26a2ff', color: '#fff', fontSize:'12px', padding:'0 15px'},
                handler: () => editSectionLoad(index)
              }
            ]"
          :title="n.stuName+' ( '+ n.stuPhone +' ) '">
        </mt-cell-swipe> 
        </div>
      </mt-tab-container-item>
      <mt-tab-container-item id="list2">
        <div class="top">
          <h4>课程列表</h4>
        </div>
        <div class="operat">
          <div class="serch">
            <input type="text">
            <mt-button type="primary" size="small">搜索</mt-button>
          </div>
          <mt-button type="primary" size="small" @click="LoadRights()">新增</mt-button>
          <mt-popup v-model="popupVisible" position="right" >
            <div class="rightContent">
              <h4>{{insertLessonList.title}}</h4>
              <mt-field label="课程名称" v-model="insertLessonList.lessonName" placeholder="请输入课程名称" ></mt-field>
              <mt-field label="课程期数" v-model="insertLessonList.lessonEpisode" placeholder="请填写课程期数" ></mt-field>
              <mt-field label="课程日期"  v-model="insertLessonList.lessonTime" placeholder="请输入课程日期" type="date"></mt-field>
              <mt-field label="备注" v-model="insertLessonList.lessonContent" placeholder="请输入备注" type="textarea" rows="4"></mt-field>
              <mt-field label="选择学生" v-if="insertLessonList.showStu" readonly  placeholder="请选择参加本课程的学生" >
                 <mt-button type="primary" size="small" @click="LoadStudentList()">选择</mt-button>
              </mt-field>
              <div class="text-center m-30">
                <mt-button type="primary" size="small" @click="insertLesson()">确定</mt-button>
                <mt-button type="primary" size="small" @click="LoadRights()">返回</mt-button>
              </div>
            </div>
            
          </mt-popup>
           <mt-popup v-model="studentVisible" position="right" >
              <div class="rightContent">
                <h4>选择参加学生</h4>
                <div class="checkList">
                  <mt-checklist
                    align="right"
                    title=""
                    v-model="value"
                    :options="options">
                  </mt-checklist>
                </div>
                 <div class="text-center m-30">
                  <mt-button type="primary" size="small" v-if="selectAllType == false" @click="selectAll()">全选</mt-button>
                  <mt-button type="primary" size="small" v-if="selectAllType == true" @click="selectAll()">全不选</mt-button>
                  <mt-button type="primary" size="small" @click="insertLessonStudentList()">确定</mt-button>
                  <mt-button type="primary" size="small" @click="LoadStudentList()">返回</mt-button>
                </div>
              </div>
              
            </mt-popup>
           <mt-popup v-model="gradeType" position="right" >
             <div class=" rightContent" v-if="temp">
               <h4>该课程学生评分列表</h4>
             <mt-cell-swipe
          :key="index"  v-for="(n,index) in gradeStatusList"
          :right="[
              {
                content: '评分',
                style: { background: '#ff7900', color: '#fff', fontSize:'12px', padding:'0 15px'},
                handler: () => loadEditgrade(index)
              },
              {
                content: '评价',
                style: { background: '#26a2ff', color: '#fff', fontSize:'12px', padding:'0 15px'},
                handler: () => loadEditevaluate(index)
              }
            ]"
          :title="n.stuName">
        </mt-cell-swipe> 
         <mt-button type="primary" size="large" @click="Loadgrade()">返回</mt-button>
        </div>
           </mt-popup>
           <mt-popup v-model="gradeListType" position="right" >
              <div class="rightContent">
                <div class="list" v-for="(grade,index) in dimensionList" :key="index">
                  <h4>{{grade.capName}}</h4>
                  <div class="checkList">
                    <mt-checklist
                      align="right"
                      title=""
                      v-model="grade.capValue"
                      :options="grade.childrenCaps">
                    </mt-checklist>
                  </div>
                </div>
                
                 <div class="text-center m-30">
                 
                  <mt-button type="primary" size="small" @click="saveGrade()">保存</mt-button>
                  <mt-button type="primary" size="small" @click="loadEditgrade()">返回</mt-button>
                </div>
              </div>
              
            </mt-popup>
        </div>
        <div class="studentList" v-if="temps">
          <mt-cell-swipe
          :key="index"  v-for="(n,index) in lessonList"
          :right="[
              {
                content: '删除',
                style: { background: '#ff7900', color: '#fff', fontSize:'12px', padding:'0 15px'},
                handler: () => delLesson(index)
              },
              {
                content: '编辑',
                style: { background: '#26a2ff', color: '#fff', fontSize:'12px', padding:'0 15px'},
                handler: () => LoadEditRights(index)
              },
              {
                content: '评分',
                style: { background: '#41b883', color: '#fff', fontSize:'12px', padding:'0 15px'},
                handler: () => Loadgrade(index)
              }
            ]"
          :title="n.lessonName+' ( '+ n.lessonTime +' )'">
        </mt-cell-swipe> 
        </div>
      </mt-tab-container-item>
    </mt-tab-container>

    <mt-tabbar v-model="selected" :fixed="fix" @click="shows()">
      <mt-tab-item id="home">
        <img slot="icon" src="../assets/logo.png">
        首页
      </mt-tab-item>
      <mt-tab-item id="page">
        <img slot="icon" src="../assets/logo.png">
        配置
      </mt-tab-item>
    
      <mt-tab-item id="list1" @click.native="shows()">
        <img slot="icon" src="../assets/logo.png">
        学生列表
      </mt-tab-item>
      
      
      <mt-tab-item id="list2" @click.native="shows()">
        <img slot="icon" src="../assets/logo.png">
        课程列表
      </mt-tab-item>
    </mt-tabbar>
  </div>
</template>

<script>
import { Button } from "mint-ui";
import { Popup } from "mint-ui";
import { Cell } from "mint-ui";
import { Field } from "mint-ui";
import { CellSwipe } from "mint-ui";
import { Checklist } from 'mint-ui';
import { Toast } from 'mint-ui';
import { MessageBox } from 'mint-ui';
import { Tabbar, TabItem, TabContainer, TabContainerItem } from "mint-ui";
// import func from '../../vue-temp/vue-editor-bridge';
export default {
  name: "Home",
  data() {
    return {
      selected: "home",
      popupVisible: false,
      studentVisible:false,
      popupStudent:false,
      ClassVisible:false,
      fix: true,
      progressNum: 3,
      latelyNum: 3,
      temp:false,
      temps:false,
      gradeType:false,
      gradeListType:false,
      selectAllType:false,
      value:[],
      options:[],
      sxjson:{
        studentId:'',
        lessonId:''
      },
      gradeStatusList:[],
      dimensionTitleList:[],
      dimensionList:[],
      studentList:[],
      lessonList:[],
      insertStudentList:{
          id:'',
          title:'',
          stuPhone:'',
          stuName:''
      },
      insertLessonList:{
        id:'',
        title:'',
        lessonEpisode:'',
        lessonName:'',
        lessonContent:'',
        lessonTime:'',
        showStu:false
      }
    };
  },
  components: {
    mtButton: Button,
    mtPopup: Popup,
    mtCell: Cell,
    mtField: Field,
    mtTabbar: Tabbar,
    mtTabItem: TabItem,
    mtCellSwipe: CellSwipe,
    mtChecklist: Checklist,
    mtTabContainer: TabContainer,
    mtTabContainerItem: TabContainerItem
  },
  updated() {
    this.$nextTick(function(){
      this.temp = true
       this.temps = true
    })
  },
  mounted() {
    this.showEcharts();
    this.showStudent();
    this.showLesson();
  },
  methods: {
    //加载学生列表
    showStudent(){
       let postData = this.$qs.stringify({
          pageNum:1,
          pageSize:100
      });
      this.$axios({
        method: 'post',
        url: 'http://192.168.1.222/course/student/queryStudentList',
        data:postData,
        }).then(function(response){
          this.studentList = response.data.data.rows
          this.options = this.studentList.map(function(item){
              return {
                label: item.stuName,
                value: item.id
              }
          });
        }.bind(this)).catch(function(error){
          console.log(error);
        });
    },
    //打开新增学生页面
    LoadAddStudent(){
     
      this.insertStudentList.id =  ''
      this.insertStudentList.title = '新增学生'
      this.insertStudentList.stuPhone = ''
      this.insertStudentList.stuName = ''
       this.popupStudent = !this.popupStudent
    },
    // 打开编辑学生页面
    editSectionLoad(index) {
       this.insertStudentList.title = '编辑学生'
       this.insertStudentList.id =  this.studentList[index].id
       this.insertStudentList.stuPhone = this.studentList[index].stuPhone
       this.insertStudentList.stuName = this.studentList[index].stuName
       this.popupStudent = !this.popupStudent
    },
    insertStudent(){
     let postData = this.$qs.stringify({
          stuPhone:this.insertStudentList.stuPhone,
          stuName:this.insertStudentList.stuName,
          id:this.insertStudentList.id
      });
      //新增学生
      // console.log(this.insertStudentList.stuPhone == '')
      if(this.insertStudentList.stuPhone == '' || this.insertStudentList.stuName == ''){
          console.log(1)
          Toast({
            message: '请填写完整信息',
            position: 'bottom',
            duration: 1000
          });
        }else{
          if(this.insertStudentList.id == ''){
            this.$axios({
            method: 'post',
            url: 'http://192.168.1.222/course/student/insertStudent',
            data:postData,
            }).then(function(response){
              Toast({
                message: '新增成功',
                position: 'bottom',
                duration: 1000
              });
              this.popupStudent = !this.popupStudent;
              this.showStudent();
            }.bind(this)).catch(function(error){
              console.log(error);
            });
          }else{
            //修改学生
            this.$axios({
            method: 'post',
            url: 'http://192.168.1.222/course/student/updateStudent',
            data:postData,
            }).then(function(response){
              Toast({
                message: '修改成功',
                position: 'bottom',
                duration: 1000
              });
              this.popupStudent = !this.popupStudent;
              this.showStudent();
            }.bind(this)).catch(function(error){
              console.log(error);
            });
          }
        }
      
      
      
    },
    //删除学生
    deleteSection(index) {
      MessageBox.confirm('确定删除此学生?').then(action => {
        let postData = this.$qs.stringify({
          id:this.studentList[index].id
        });
        this.$axios({
          method: 'post',
          url: 'http://192.168.1.222/course/student/delStudent',
          data:postData,
          }).then(function(response){
            Toast({
              message: '删除成功',
              position: 'bottom',
              duration: 1000
            });
            this.showStudent();
          }.bind(this)).catch(function(error){
            console.log(error);
          });
        });
    },

    //加载课程列表
    showLesson(){
      let postData = this.$qs.stringify({
          pageNum:1,
          pageSize:100
      });
      this.$axios({
        method: 'post',
        url: 'http://192.168.1.222/course/lesson/queryLessonList',
        data:postData,
        }).then(function(response){
           
          this.lessonList = response.data.data.rows
        }.bind(this)).catch(function(error){
          console.log(error);
        });
    },
    
    LoadClassList(){
      this.ClassVisible = !this.ClassVisible
    },
    //打开新增课程
    LoadRights(){
      this.insertLessonList.showStu= false
      this.insertLessonList.id = ''
      this.insertLessonList.title='新增课程'
      this.insertLessonList.lessonEpisode=''
      this.insertLessonList.lessonName=''
      this.insertLessonList.lessonContent=''
      this.insertLessonList.lessonTime=''
      this.popupVisible = !this.popupVisible
    },
    //打开编辑课程
    LoadEditRights(index){
      this.insertLessonList.showStu= true
      this.insertLessonList.id = this.lessonList[index].id
      this.insertLessonList.title='编辑课程'
      this.insertLessonList.lessonEpisode= this.lessonList[index].lessonEpisode
      this.insertLessonList.lessonName= this.lessonList[index].lessonName
      this.insertLessonList.lessonContent= this.lessonList[index].lessonContent
      this.insertLessonList.lessonTime= this.lessonList[index].lessonTime
      this.popupVisible = !this.popupVisible
    },
    insertLesson(){
      let postData = this.$qs.stringify(this.insertLessonList);
     
      if(this.insertLessonList.lessonEpisode == '' || this.insertLessonList.lessonName == '' || this.insertLessonList.lessonContent == '' || this.insertLessonList.lessonTime == ''){
        Toast({
            message: '请填写完整信息',
            position: 'bottom',
            duration: 1000
          });
      }else{
          //新增课程
        if(this.insertLessonList.title == '新增课程'){
          this.$axios({
            method: 'post',
            url: 'http://192.168.1.222/course/lesson/insertLesson',
            data:postData,
            }).then(function(response){
              Toast({
                message: '新增成功',
                position: 'bottom',
                duration: 1000
              });
              this.popupVisible = !this.popupVisible;
              this.showLesson();
            }.bind(this)).catch(function(error){
              console.log(error);
            });
        }else{
          //编辑课程
            this.$axios({
            method: 'post',
            url: 'http://192.168.1.222/course/lesson/updateLesson',
            data:postData,
            }).then(function(response){
              Toast({
                message: '编辑成功',
                position: 'bottom',
                duration: 1000
              });
              this.popupVisible = !this.popupVisible;
              this.showLesson();
            }.bind(this)).catch(function(error){
              console.log(error);
            });
        }
      }
    },
    //删除课程
    delLesson(index){
      MessageBox.confirm('确定删除此课程?').then(action => {
        let postData = this.$qs.stringify({
          id:this.lessonList[index].id
        });
        this.$axios({
          method: 'post',
          url: 'http://192.168.1.222/course/lesson/delLesson',
          data:postData,
          }).then(function(response){
            Toast({
              message: '删除成功',
              position: 'bottom',
              duration: 1000
            });
            this.showLesson();
          }.bind(this)).catch(function(error){
            console.log(error);
          });
        });
    },
    //打开选择学生列表
    LoadStudentList(){
      this.studentVisible = !this.studentVisible
      this.queryLessonStudentList();
    },
    //全选学生
    selectAll(){
      this.selectAllType = !this.selectAllType
      if(this.selectAllType == false){
        this.value =[]
      }else{
        this.value = this.studentList.map(function(item){
          return item.id
        })
      }
      
    },
    //获取课程学生列表
    queryLessonStudentList(){
      let postData = this.$qs.stringify({
          lessonId:this.insertLessonList.id,
          pageNum:1,
          pageSize:1000
        });
      this.$axios({
        method: 'post',
        url: 'http://192.168.1.222/course/lessonStudent/queryLessonStudentList',
        data:postData,
        }).then(function(response){
          if(response.data.success){
            this.value = response.data.data.rows.map(function(item){
              return item.studentId
            })
          }else{
            this.value = []
          }
          
        }.bind(this)).catch(function(error){
          console.log(error);
        });
    },
    //课程选择学生列表
    insertLessonStudentList(){
      var ms = [];
      ms = this.value.map(function(item){
         return {
           studentId:item
         }
      })
      let postData = this.$qs.stringify({
          studentIds:JSON.stringify(ms),
          lessonId:this.insertLessonList.id
        });
      this.$axios({
        method: 'post',
        url: 'http://192.168.1.222/course/lessonStudent/insertLessonStudentList',
        data:postData,
        }).then(function(response){
          Toast({
            message: '添加学生成功',
            position: 'bottom',
            duration: 1000
          });
          this.studentVisible = !this.studentVisible;
          this.showLesson();
        }.bind(this)).catch(function(error){
          console.log(error);
        });
    },
    //打开评分
    Loadgrade(index){
      this.shows()
      this.gradeType = !this.gradeType;
      if(index!=''||index!=null){
        this.sxjson.lessonId = this.lessonList[index].id
      let postData = this.$qs.stringify({
          lessonId:this.sxjson.lessonId
        });
        this.$axios({
          method: 'post',
          url: 'http://192.168.1.222/course/lessonStudent/queryLessonStudentList',
          data:postData,
          }).then(function(response){
           
            this.gradeStatusList = response.data.data.rows
          }.bind(this)).catch(function(error){
            console.log(error);
          });
      }
      
    },
    //打开能力维度评分列表
    loadEditgrade(index){
      this.loadDimension();
      this.gradeListType = !this.gradeListType
      if(index!=''||index!=null){
        this.sxjson.studentId = this.gradeStatusList[index].studentId
      }
      
      
    },
    //获取能力维度
    loadDimension(){
      this.$axios({
        method: 'post',
        url: 'http://192.168.1.222/course/capability/queryCapabilityTree'
        }).then(function(response){
          this.dimensionList = response.data.data.map(function(item){
            return{
              capName:item.capName,
              id:item.id,
              capValue:[],
              childrenCaps:item.childrenCaps.map(function(i){
                return {
                  label:i.capName,
                  value:i.id,
                  pid:item.id
                }
              })
            }
          })
        }.bind(this)).catch(function(error){
          console.log(error);
        });
    },
    //保存维度评分
    saveGrade(){
     var newArr = this.dimensionList.reduce((total, i) => {
          return total.concat(i.capValue.map(item => ({
          capabilityId: item,
          parentCapId: i.id,
          score:100
          })));
        }, []);
      let postData = this.$qs.stringify({
          studentId:this.sxjson.studentId,
          userId:'1',
          lessonId:this.sxjson.lessonId,
          scoreJson:JSON.stringify(newArr) 
        });
        // console.log(this.dimensionList)
        this.$axios({
          method: 'post',
          url: 'http://192.168.1.222/course/score/insertScore',
          data:postData,
          }).then(function(response){
            Toast({
              message: '评分成功',
              position: 'bottom',
              duration: 1000
            });
            this.gradeListType = !this.gradeListType
          }.bind(this)).catch(function(error){
            console.log(error);
          });
    },
   shows(){
      this.temp = false
      this.temps = false
      this.$nextTick(function(){
        this.temp = true
        this.temps = true
      })
   },
   
    
    // 查看学生
    checkSection(index) {
      console.log(index);
    },
    showEcharts() {
      let myChart1 = this.$echarts.init(document.getElementById("studentSum"));
      let myChart2 = this.$echarts.init(document.getElementById("section"));
      let myChart3 = this.$echarts.init(document.getElementById("dimension"));
      let myChart4 = this.$echarts.init(document.getElementById("progress"));
      let myChart5 = this.$echarts.init(document.getElementById("lately"));
      let option1 = {
        tooltip: {
          trigger: "axis"
        },
        legend: {
          data: ["学生总数", "本期学生总数"]
        },
        toolbox: {
          show: true,
          feature: {
            magicType: { show: true, type: ["line", "bar"] }
          }
        },
        calculable: true,
        xAxis: [
          {
            type: "category",
            boundaryGap: false,
            data: ["周一", "周二", "周三", "周四", "周五", "周六", "周日"]
          }
        ],
        yAxis: [
          {
            type: "value"
          }
        ],
        series: [
          {
            name: "本期学生总数",
            type: "line",
            stack: "总量",
            data: [120, 132, 101, 134, 90, 230, 210]
          },
          {
            name: "学生总数",
            type: "line",
            stack: "总量",
            data: [220, 182, 191, 234, 290, 330, 310]
          }
        ]
      };
      let option2 = {
        tooltip: {
          trigger: "item",
          formatter: "{a} <br/>{b} : {c} ({d}%)"
        },
        toolbox: {
          show: true
        },
        // calculable: true,
        series: [
          {
            name: "满分100分",
            type: "pie",
            radius: "55%",
            // center: ["50%", "60%"],
            data: [
              { value: 335, name: "70分以下" },
              { value: 310, name: "70分-79分" },
              { value: 234, name: "80分-89分" },
              { value: 135, name: "90分以上" }
            ]
          }
        ]
      };
      let option3 = {
        tooltip: {
          trigger: "item",
          formatter: "{a} <br/>{b} : {c} ({d}%)"
        },
        toolbox: {
          show: true
        },
        // calculable: true,
        series: [
          {
            name: "满分",
            type: "pie",
            radius: "55%",
            // center: ["50%", "60%"],
            data: [
              { value: 335, name: "沟通能力" },
              { value: 310, name: "领导能力" },
              { value: 234, name: "适应性" },
              { value: 135, name: "思维习惯" },
              { value: 300, name: "决策能力" },
              { value: 234, name: "批判性思维" },
              { value: 135, name: "分析创造" },
              { value: 300, name: "全球视野" }
            ]
          }
        ]
      };
      let option4 = {
        tooltip: {
          trigger: "item",
          formatter: "{a} <br/>{b} : {c} ({d}%)"
        },

        legend: {
          x: "center",
          data: ["小明（沟通能力）", "小红（表达能力）", "小王（领导能力）"]
        },
        toolbox: {
          show: true
        },
        // calculable: true,
        series: [
          {
            name: "满分100分",
            type: "pie",
            radius: "55%",
            itemStyle: {
              normal: {
                label: {
                  show: false //隐藏标示文字
                },
                labelLine: {
                  show: false //隐藏标示线
                }
              }
            },
            // center: ["50%", "60%"],
            data: [
              { value: 335, name: "小明（沟通能力）" },
              { value: 310, name: "小红（表达能力）" },
              { value: 234, name: "小王（领导能力）" }
            ]
          }
        ]
      };
      let option5 = {
        tooltip: {
          trigger: "item",
          formatter: "{a} <br/>{b} : {c} ({d}%)"
        },

        legend: {
          x: "center",
          data: ["小明（沟通能力）", "小红（表达能力）", "小王（领导能力）"]
        },
        toolbox: {
          show: true
        },
        // calculable: true,
        series: [
          {
            name: "满分100分",
            type: "pie",
            radius: "55%",
            itemStyle: {
              normal: {
                label: {
                  show: false //隐藏标示文字
                },
                labelLine: {
                  show: false //隐藏标示线
                }
              }
            },
            // center: ["50%", "60%"],
            data: [
              { value: 335, name: "小明（沟通能力）" },
              { value: 310, name: "小红（表达能力）" },
              { value: 234, name: "小王（领导能力）" }
            ]
          }
        ]
      };
      myChart1.setOption(option1);
      myChart2.setOption(option2);
      myChart3.setOption(option3);
      myChart4.setOption(option4);
      myChart5.setOption(option5);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.mint-popup-right{
  overflow-y: auto;
}
.text-center{
  text-align: center;
}
.m-30{
  margin-top: 30px;
}
.home {
  height: 100%;
  overflow: auto;
  padding: 15px;
  padding-bottom: 60px;
  box-sizing: border-box;
}
.home .top {
  background-color: #ddd;
  padding: 15px;
}
.home .top h4 {
  font-size: 22px;
}
.mint-tabbar > .mint-tab-item.is-selected {
  color: #000;
}
.configure {
  margin-top: 15px;
}
.configure .mint-field .mint-cell-title {
  flex: 1;
}
.configure .mint-cell-wrapper,
.configure .mint-cell:last-child {
  background: #fff!important;
}
.save {
  margin-top: 15px;
  text-align: center;
}
.item {
  background-color: #f5f5f5;
  padding: 15px;
  margin: 10px 0;
}
.echart {
  width: 100%;
  padding: 10px;
  box-sizing: border-box;
  height: 300px;
}
.operat {
  margin-top: 15px;
}
.operat,
.operat .serch {
  display: flex;
}
.operat .serch {
  flex-grow: 1;
}
.operat .serch button {
  border-radius: 0;
}
.operat .serch input {
  padding: 0 5px;
  outline: none;
  border: 1px solid #ccc;
  border-right: 0;
}
.studentList {
  margin-top: 15px;
  background-color: #eee;
}
.studentList table {
  width: 100%;
  text-align: center;
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
.mint-checklist .mint-cell-title{
  width: 100%;
}
.mint-toast.is-placebottom{
  z-index: 10000;
}
.mint-cell-value input{
  background-color: #fff;
}
</style>
