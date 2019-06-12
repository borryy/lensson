<template>
  <div class="home">
    <mt-tab-container v-model="selected">
      <mt-tab-container-item id="home">
        <div class="top">
          <h4>研学学分数据统计</h4>
        </div>
        <div class="studentSum item">
          <h3>学生总数 
            
            </h3>
          <div id="studentSum" class="echart"></div>
        </div>
        <div class="section item">
          <h3>分数区间数量分布 
           <select v-model="fid" @change="queryecharts1()" style="padding:5px;outline:none;">
             <option v-for="(item,index) in dimensionList" :key="index" :value="item.id">{{item.capName}}</option>
           </select>
          </h3>
          <div id="section" class="echart"></div>
        </div>
        <div class="dimension item">
          <h3>各维度满分学生分布</h3>
          <div id="dimension" class="echart"></div>
        </div>
       
      </mt-tab-container-item>
      <mt-tab-container-item id="page">
        <div class="top">
          <h4>Top排行榜</h4>
        </div>
        <div class="configure">
          <h3>进步最快的学生Top{{zkxs}}
            <select v-model="zkxs" style="padding:5px;outline:none;" @change="queryecharts3()">
              <option value="3">前三名</option>
              <option value="5">前五名</option>
              <option value="10">前十名</option>
              <option value="20">前二十名</option>
            </select>
          </h3>
          <div class="listzk">
            <mt-cell  title="姓名" value="进步分数"></mt-cell>
            <mt-cell v-for="(item,index) in rankByImprove" :key="index" :title="item.stuName" :value="item.scoreImproved"></mt-cell>
          </div>
          <h3>课程各维度评分排名Top{{mfxs}}
            <select v-model="mfxs" style="padding:5px;outline:none;" @change="queryecharts4()">
              <option value="3">前三名</option>
              <option value="5">前五名</option>
              <option value="10">前十名</option>
              <option value="20">前二十名</option>
            </select>
          </h3>
          <div class="listzk">
            <mt-cell  title="姓名" value="分数"></mt-cell>
            <mt-cell v-for="(item,index) in rankByCapabilityPid" :key="index" :title="item.stuName" :value="item.scoreTotal"></mt-cell>
          </div>
        </div>
        
      </mt-tab-container-item>
      <mt-tab-container-item id="list1">
        <div class="top">
          <h4>学生列表</h4>
        </div>
        <div class="operat">
          <div class="serch">
            <input type="text" v-model="stuNames">
            <mt-button type="primary" size="small" @click="showStudent(stuNames)">搜索</mt-button>
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
            <input type="text" v-model="lessonTimes">
            <mt-button type="primary" size="small" @click="showLesson(lessonTimes)">搜索</mt-button>
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
         <mt-button type="primary" size="large" @click="Loadgrades()">返回</mt-button>
        </div>
           </mt-popup>
            <mt-popup v-model="pjType" position="right" >
             <div class=" rightContent" v-if="temp">
              <h4>该课程学生评价</h4>
              <textarea style="width:100%;outline:none;padding:10px;box-sizing:border-box;" name="" id="" cols="30" rows="10" v-model="pj"></textarea>
              <div style="text-align:center;">
                <mt-button type="primary" size="small" @click="savepj()">确定</mt-button>
                <mt-button type="primary" size="small" @click="closepj()">返回</mt-button>
              </div>
              
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
                  <mt-button type="primary" size="small" @click="loadEditgrades()">返回</mt-button>
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
        <img slot="icon" src="../assets/home.png">
        首页
      </mt-tab-item>
      <mt-tab-item id="page">
        <img slot="icon" src="../assets/top.png">
        Top榜
      </mt-tab-item>
    
      <mt-tab-item id="list1" @click.native="shows()">
        <img slot="icon" src="../assets/student.png">
        学生列表
      </mt-tab-item>
      <mt-tab-item id="list2" @click.native="shows()">
        <img slot="icon" src="../assets/lesson.png">
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
import { Picker } from 'mint-ui';
import { MessageBox } from 'mint-ui';
import { Tabbar, TabItem, TabContainer, TabContainerItem } from "mint-ui";
// import func from '../../vue-temp/vue-editor-bridge';
export default {
  name: "Home",
  data() {
    return {
      fid:'1',
      fids:'1',
      stuNames:'',
      lessonTimes:'',
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
      pjType:false,
      gradeListType:false,
      selectAllType:false,
      value:[],
      options:[],
      sxjson:{
        studentId:'',
        lessonId:''
      },
      pj:'',
      pjid:'',
      voluntarily:'',
      gradeStatusList:[],
      dimensionTitleList:[],
      dimensionList:[],
      studentList:[],
      lessonList:[],
      CountStudent:{},
      countStudentByScore:{},
      rankByImprove:{},
      rankByCapabilityPid:{},
      countFullScoreByLessonId:{},
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
      },
      zkxs:'3',
      mfxs:'3'
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
    mtPicker:Picker,
    mtTabContainer: TabContainer,
    mtTabContainerItem: TabContainerItem
  },
  updated() {
    this.$nextTick(function(){
      this.temp = true
      this.temps = true
      
    })
    // this.showEcharts();
  },
  mounted() {
    if(this.$route.params.msg != "success"){
      this.$router.push({
            name: 'Login',
            params: {
              
            }
        })
    }
    
    this.showStudent(this.stuNames);
    this.showLesson();
    this.queryecharts();
    this.queryecharts1();
    this.queryecharts2();
    this.queryecharts3();
    this.queryecharts4();
    this.loadDimension();
  },
  methods: {
    //加载学生列表
    showStudent(stuName){
       let postData = this.$qs.stringify({
          pageNum:1,
          pageSize:100,
          stuName:stuName
      });
      this.$axios({
        method: 'post',
        url: '/course/student/queryStudentList',
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
            url: '/course/student/insertStudent',
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
            url: '/course/student/updateStudent',
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
          url: '/course/student/delStudent',
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
    showLesson(lessonName){
      let postData = this.$qs.stringify({
          pageNum:1,
          pageSize:100,
          lessonName:lessonName
      });
      this.$axios({
        method: 'post',
        url: '/course/lesson/queryLessonList',
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
            url: '/course/lesson/insertLesson',
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
            url: '/course/lesson/updateLesson',
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
          url: '/course/lesson/delLesson',
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
        url: '/course/lessonStudent/queryLessonStudentList',
        data:postData,
        }).then(function(response){
          if(response.data.success){
            this.value = []
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
      
      // var ms = [];
      // this.value = this.value.map(function(item){
      //    return {
      //      studentId:item
      //    }
      // })
      this.value = this.value.map(item => ({
          studentId: item
        })).sort((prev, next) => {
          return prev.studentId>next.studentId;
        }).filter((item, index, arr) => {
          if(index == 0){
            return true
          }else{
            return item.studentId != arr[index-1].studentId;
          }
          
        });
      console.log(this.value)
      let postData = this.$qs.stringify({
          studentIds:JSON.stringify(this.value),
          lessonId:this.insertLessonList.id
        });
      this.$axios({
        method: 'post',
        url: '/course/lessonStudent/insertLessonStudentList',
        data:postData,
        }).then(function(response){
          Toast({
            message: '添加学生成功',
            position: 'bottom',
            duration: 1000
          });
          this.value=[]
          this.studentVisible = !this.studentVisible;
          this.showLesson();
        }.bind(this)).catch(function(error){
          console.log(error);
        });
    },
    //关闭评分
    Loadgrades(){
      this.gradeType = !this.gradeType;
    },
    //打开评分
    Loadgrade(index){
      this.shows()
      this.gradeType = !this.gradeType;
      this.sxjson.lessonId = this.lessonList[index].id
      let postData = this.$qs.stringify({
          lessonId:this.sxjson.lessonId
        });
      this.$axios({
        method: 'post',
        url: '/course/lessonStudent/queryLessonStudentList',
        data:postData,
        }).then(function(response){
          this.gradeStatusList = response.data.data.rows
        }.bind(this)).catch(function(error){
          console.log(error);
        });
      
    },
    //关闭能力维度评分列表
    loadEditgrades(){
      this.gradeListType = !this.gradeListType
    },
    //打开能力维度评分列表
    loadEditgrade(index){
      this.loadDimension();
      this.gradeListType = !this.gradeListType
      this.sxjson.studentId = this.gradeStatusList[index].studentId
    },
    //获取能力维度
    loadDimension(){
      this.$axios({
        method: 'post',
        url: '/course/capability/queryCapabilityTree'
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
          console.log(this.dimensionList)
        }.bind(this)).catch(function(error){
          console.log(error);
        });
    },
    //保存维度评分
    saveGrade(){
     var newArr = this.dimensionList.reduce((total, i) => {
          return total.concat(i.capValue.map(item => ({
          capabilityId: item,
          parentCapid: i.id,
          score:100
          })));
        }, []);
      let postData = this.$qs.stringify({
          studentId:this.sxjson.studentId,
          userId:'1',
          lessonId:this.sxjson.lessonId,
          scoreJson:JSON.stringify(newArr) 
        });
        this.$axios({
          method: 'post',
          url: '/course/score/queryScoreList',
          data:postData,
          }).then(function(response){
            if(response.data.data.length < 1){
              this.$axios({
              method: 'post',
              url: '/course/score/insertScore',
              data:postData,
              }).then(function(response){
                Toast({
                  message: '评分成功',
                  position: 'bottom',
                  duration: 1000
                });
                this.makeAppraise();
                
                this.gradeListType = !this.gradeListType
              }.bind(this)).catch(function(error){
                console.log(error);
              });
            }else{
              this.$axios({
              method: 'post',
              url: '/course/score/updateScoreDetail',
              data:postData,
              }).then(function(response){
                Toast({
                  message: '评分修改成功',
                  position: 'bottom',
                  duration: 1000
                });
                this.makeAppraise();
                
                this.gradeListType = !this.gradeListType
              }.bind(this)).catch(function(error){
                console.log(error);
              });
            }
          }.bind(this)).catch(function(error){
            console.log(error);
          });
        // this.$axios({
        //   method: 'post',
        //   url: '/course/score/insertScore',
        //   data:postData,
        //   }).then(function(response){
        //     Toast({
        //       message: '评分成功',
        //       position: 'bottom',
        //       duration: 1000
        //     });
        //     this.makeAppraise();
            
        //     this.gradeListType = !this.gradeListType
        //   }.bind(this)).catch(function(error){
        //     console.log(error);
        //   });
    },
    //自动生成评价
    makeAppraise(){
      let postData = this.$qs.stringify({
          studentId:this.sxjson.studentId,
          lessonId:this.sxjson.lessonId
        });
        this.$axios({
          method: 'post',
          url: '/course/comment/autoComment',
          data:postData,
          }).then(function(response){
            var str = '';
            for(var i=0;i<response.data.data.length;i++){
              if(response.data.data[i]!=null){
                str+=response.data.data[i].contentDetail+','
              }else{
                str+=''
              }
              
            }
            this.voluntarily = str;
            console.log(this.voluntarily)
            this.addAppraise(this.voluntarily)
          }.bind(this)).catch(function(error){
            console.log(error);
          });
    },
    //查看评价
    queryAppraise(){
      let postData = this.$qs.stringify({
          studentId:this.sxjson.studentId,
          lessonId:this.sxjson.lessonId
        });
        this.$axios({
          method: 'post',
          url: '/course/comment/queryCommentByLessonIdOrStudentId',
          data:postData,
          }).then(function(response){
            this.pj = response.data.data.content
            this.pjid= response.data.data.id
          }.bind(this)).catch(function(error){
            console.log(error);
          });
    },
    //打开评价
    loadEditevaluate(index){
        this.sxjson.studentId = this.gradeStatusList[index].studentId
        this.queryAppraise()
        this.pjType = !this.pjType
    },
    //关闭评价
    closepj(){
      this.pjType = !this.pjType
    },
    //修改评价
    savepj(){
       let postData = this.$qs.stringify({
          id:this.pjid,
          content:this.pj
        });
        this.$axios({
          method: 'post',
          url: '/course/comment/updateComment',
          data:postData,
          }).then(function(response){
            Toast({
              message: '评价成功',
              position: 'bottom',
              duration: 2000
            });
            this.pjType = !this.pjType
          }.bind(this)).catch(function(error){
            console.log(error);
          });
    },
    //添加评价
    addAppraise(voluntarily){
      let postData = this.$qs.stringify({
          studentId:this.sxjson.studentId,
          lessonId:this.sxjson.lessonId,
          content:voluntarily
        });
        this.$axios({
          method: 'post',
          url: '/course/comment/insertComment',
          data:postData,
          }).then(function(response){
            Toast({
              message: '评价成功',
              position: 'bottom',
              duration: 2000
            });
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
   //获取echarts
   queryecharts(){
     let postData = this.$qs.stringify({
          pageSize:7
        });
        this.$axios({
          method: 'post',
          url: '/course/echarts/CountStudent',
          data:postData,
          }).then(function(response){
            this.CountStudent = response.data.data
            this.showEcharts1()
          }.bind(this)).catch(function(error){
            console.log(error);
          });
   },
   queryecharts1(){
   
      let postData = this.$qs.stringify({
          capabilityPid:this.fid
        });
    
        this.$axios({
          method: 'post',
          url: '/course/echarts/countStudentByScore',
          data:postData,
          }).then(function(response){
            this.countStudentByScore = response.data.data
            this.showEcharts2()
          }.bind(this)).catch(function(error){
            console.log(error);
          });
   },
   
   queryecharts2(){
     let postData = this.$qs.stringify({
          // pageSize:7
        });
        this.$axios({
          method: 'post',
          url: '/course/echarts/countFullScoreByLessonId',
          data:postData,
          }).then(function(response){
            this.countFullScoreByLessonId = response.data.data
            this.showEcharts3()
          }.bind(this)).catch(function(error){
            console.log(error);
          });
   },
   queryecharts3(){
     let postData = this.$qs.stringify({
          pageSize:this.zkxs
        });
        this.$axios({
          method: 'post',
          url: '/course/echarts/rankByImprove',
          data:postData,
          }).then(function(response){
            this.rankByImprove = response.data.data.rows
            // this.showEcharts4()
          }.bind(this)).catch(function(error){
            console.log(error);
          });
   },
   queryecharts4(){
     let postData = this.$qs.stringify({
          capabilityPid:this.fids
        });
        this.$axios({
          method: 'post',
          url: '/course/echarts/rankByCapabilityPid',
          data:postData,
          }).then(function(response){
            this.rankByCapabilityPid = response.data.data.rows
            // this.showEcharts5()
          }.bind(this)).catch(function(error){
            console.log(error);
          });
   },
   showEcharts1() {
     let that = this;
     let myChart1 = this.$echarts.init(document.getElementById("studentSum"));
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
            data: that.CountStudent.lessonTime
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
            data: that.CountStudent.lessonStudents
          },
          {
            name: "学生总数",
            type: "line",
            stack: "总量",
            data: that.CountStudent.studentSum
          }
        ]
      };
    myChart1.setOption(option1);
   },
   showEcharts2() {
      var that = this;
      let myChart2 = this.$echarts.init(document.getElementById("section"));
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
            data: this.countStudentByScore
          }
        ]
      };
      myChart2.setOption(option2);
   },
   showEcharts3() {
      var that = this;
      let myChart3 = this.$echarts.init(document.getElementById("dimension"));
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
            data: this.countFullScoreByLessonId
          }
        ]
      };
      myChart3.setOption(option3);
   },
  unique5(arr){
      var x = new Set(arr);
    return [...x];
    }
  },
  
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
