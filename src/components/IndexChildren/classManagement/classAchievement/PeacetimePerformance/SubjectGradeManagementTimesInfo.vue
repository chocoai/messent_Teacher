// 学生成绩表信息

/*
 * File: PerformanceDivisionManagement.vue
 * Project: teacher
 * File Created: Monday, 5th November 2018 5:53:30 pm
 * Author: LGH (you@you.you)
 * -----
 * Last Modified: Friday, 23rd November 2018 4:16:58 pm
 * Modified By: LGH (you@you.you>)
 * -----
 * Copyright 2017 - 2018 Your Company, Your Company
 * 普通教师平时成绩管理
 */

 <template>
  <div id="PerformanceDivisionManagement">
    <div class="seach">
      <div class="seach_left">
        <el-input placeholder="请输姓名,学号关键字(回车进行搜索)" suffix-icon="el-icon-search" v-model="keywordsVal" clearable @change="keyWordsInput"></el-input>
      </div>
      <div class="seach_right">
        <el-button icon="" class="seach_yellow" round @click="leadingOut">成绩导出</el-button>
        <el-button icon="" type="primary" round @click="dialogVisible2=true">成绩发送对应家长</el-button>
      </div>
    </div>
    <el-table :data="tableData.data" style="width: 100%" height="600" :header-cell-style="{textAlign:'center'}" :cell-style="{textAlign:'center',color:'#606266'}">
      <el-table-column prop="gradeClassName" label="年级班级" width="100"></el-table-column>
      <el-table-column prop='studentNo' label='学号' :show-overflow-tooltip="true"></el-table-column>
      <el-table-column prop="studentName" label="学生姓名" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column prop="subjectName" label="科目"></el-table-column>
      <el-table-column prop="examName" label="平时成绩名称" width="110"></el-table-column>
      <el-table-column prop="examType" label="平时成绩类型" width="110"></el-table-column>
      <el-table-column prop="fullScore" label="满分值"></el-table-column>

      <el-table-column prop="score" label="分数" width="110">
        
        <template slot-scope="scope">
            <span v-if="!scope.row.hoverShow" :style="{color:scope.row.examStatus==0?'':'red'}">{{scope.row.examStatus==0?scope.row.score:'缺考'}}</span>
            <el-input v-if="scope.row.hoverShow" type="number" min="0" :max="scope.row.fullScore" v-model="scope.row.score" placeholder="请输入修改后的分数"></el-input>
        </template>
      </el-table-column>
      <el-table-column prop="classRanking" label="班级排名"></el-table-column>
      <el-table-column prop="content" label="评价内容" width="120" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column label="操作" width="280px" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" @click="edit(scope.row)">{{scope.row.hoverShow?'取消':'修改成绩'}}</el-button>
          <el-button v-show="scope.row.hoverShow" type="text" @click="editOk(scope.row)">完成</el-button>
          <el-button type="text" @click="countLookMe(scope.row)">评价学生</el-button>
          <el-button class="look" type="text" @click="LookMeHistory(scope.row)">科目历史成绩</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 评价学生 -->
    <el-dialog :visible.sync="dialogVisible" :close-on-click-modal="false" top="30vh" width="34%">
      <div slot="title" style="text-align:center;">
        评价学生
      </div>
      <div class="" style="">
        <el-form ref="form" :model="form" label-width="80px">
          <el-input type="textarea" resize="none" rows="10" v-model="form.content" placeholder="请输入评价内容（200字符以内)" maxlength="200"></el-input>
        </el-form>
        <div slot="footer" style='text-align:center;padding: 20px;'>
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="onSubmit('form')">确 定</el-button>
        </div>
      </div>
    </el-dialog>

    <!-- 成绩发送对应家长 -->
    <el-dialog :visible.sync="dialogVisible2" top="30vh" :close-on-click-modal="false" width="30%">
      <div slot="title" style="text-align:center;">
        是否将{{tableData.data?tableData.data[0].subjectName:null}}成绩和评价内容发送给家？包括以下内容：
      </div>
      <div class="content" style="text-align:center;">
        <el-checkbox v-for="(item,index) in sendScores" v-model="item.flag" :label="item.label" @change="CheckChange($event,item,index)" :key="index"></el-checkbox>
      </div>
      <div slot="footer" style='padding-top: 20px;display:flex;justify-content: space-around;'>
        <el-button @click="dialogVisible2 = false">关闭</el-button>
        <el-button type="primary" @click="dialogVisible2_1=true;timing=new Date()">定时发送</el-button>
        <el-button type="primary" @click="sendMark($event)">发送</el-button>
      </div>

      <el-dialog
            width="30%"
            top="35vh"
            :visible.sync="dialogVisible2_1"
            append-to-body>
        <div slot="title" style="text-align:center;">
          定时发送
        </div>
        <div class="content" style="">
          <div class="demo-input-suffix">
            <div>选择定时发送的时间：</div> 
             <el-date-picker
              style="width:100%; margin:10px 0"
              v-model="timing"
              type="datetime"
              value-format="timestamp"
              placeholder="请选择定时发送时间"
              default-time="12:00:00" >
            </el-date-picker>
            <!-- {{timing}} -->
            <div v-if="timing">成绩将会在<span style="font-weight:600;padding:0 5px">{{new Date(timing).format('yyyy-MM-dd hh:mm:ss')}}</span>发送给家长</div>
          </div>
        </div>
        <div slot="footer" style='padding-top: 20px;display:flex;justify-content: space-around;'>
            <el-button type="primary" @click="sendMark($event,timing)">发送</el-button>
            <el-button @click="dialogVisible2_1=false">关闭</el-button>
        </div>
      </el-dialog>

    </el-dialog>

    <pageHtml :tabObj.sync='tableData' name="classStudentSubjectScoreClassStudentSubjectScoreList" :otherType='queryObj' :searchsKey='keywordsVal' :discriminate='2' :edit='true'></pageHtml>
  </div>
</template>

<script>
import {
  classStudentSubjectScoreClassStudentSubjectScoreList,
  classStudentSubjectScoreUpdateScore,
  classStudentSubjectScoreEvaluateStudent, //科目成绩评价学生
  classStudentSubjectScoreGetEvaluateContent, //获取评价
  classStudentSubjectScoreSendScore,
  classStudentSubjectScoreSendScoreByQuartzTime
} from '@/api/api'
import Vue from 'vue'
import axios from 'axios'

export default {
  inject: ['reload'],
  data() {
    return {
      //关键字过滤
      queryObj: this.$route.query,
      keywordsVal: '',
      tableData: [],
      id: '',
      dialogVisible: false,
      dialogVisible2: false,
      dialogVisible2_1:false,
      timing:new Date(),

      //评价内容
      form: { content: '' },
      sendScores: [
        // subjectId: this.$route.query.subjectId,
        {
          flag: true,
          label: '班级平均分',
          averageScore: 1
        },
        {
          flag: true,
          label: '班级排名',
          classRanking: 1
        }
      ],
      tempNumber:''
    }
  },
  methods: {
    keyWordsInput(val) {
      this.keywordsVal = val
      classStudentSubjectScoreClassStudentSubjectScoreList({
        classId: this.queryObj.classId,
        type: 2,
        term: this.queryObj.term,
        examType: this.queryObj.examType,
        subjectId: this.queryObj.subjectId,
        examName:this.queryObj.examName,
        searchWord: val
      }).then(res => {
        if (res.data.code == 200) {
          console.log(res.data.data)
          res.data.data.data.forEach(item => {
                item.hoverShow = false
          })
          this.tableData = res.data.data

        } else {
          console.log(res)
        }
      })
    },
    countLookMe(row) {
      // console.log('row', row)
      this.id = row.id
      this.dialogVisible = true
      //获取科目成绩评价学生
      classStudentSubjectScoreGetEvaluateContent({
        id: row.id
      }).then(res => {
        if (res.data.code == 200) {
          // console.log(res.data.data)
          this.form.content = res.data.data.content
        }
      })
    },
    //评价提交
    onSubmit(form) {
      //这里执行ajax
      this.$refs[form].validate(valid => {
        if (valid) {
          console.log('content: ', this.form.content, 'Id:', this.id)
          classStudentSubjectScoreEvaluateStudent({
            //编辑个人评价
            id: this.id,
            content: this.form.content
          }).then(res => {
            if (res.data.code == 200) {
              // console.log(res)
              this.$message({
                type: 'success',
                message: res.data.message
              })
              setTimeout(() => {
                this.dialogVisible = false
                this.reload()
              }, 500)
            } else {
              this.$message({
                type: 'error',
                message: res.data.message
              })
            }
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },

    //导出成绩
    leadingOut() {
      axios({
        url:
          '/studentScoreExport/exportStudentSubjectScoreExportData?classId=' +
          this.queryObj.classId +
          '&subjectId=' +
          this.queryObj.subjectId +
          '&term=' +
          this.queryObj.term +
          '&examType=' +
          this.queryObj.examType +
          '&examName=' +
          this.queryObj.examName +
          '&type=2',
        method: 'get',
        headers: { Authorization: TOKEN },
        responseType: 'blob'
      })
        .then(function(res) {
          var blob = new Blob([res.data], { type: res.headers['content-type'] })
          var downloadElement = document.createElement('a')
          var href = window.URL.createObjectURL(blob) //创建下载的链接
          downloadElement.href = href
          downloadElement.download = '学生平时科目成绩数据表.xls' //下载后文件名
          document.body.appendChild(downloadElement)
          downloadElement.click() //点击下载
          document.body.removeChild(downloadElement) //下载完成移除元素
          window.URL.revokeObjectURL(href) //释放掉blob对象
        })
        .catch(function() {})
    },

    edit(row) {
      if (!row.hoverShow) {
        this.tempNumber = row.score
        row.hoverShow = true
      } else {
        row.score = this.tempNumber
        row.hoverShow = false
      }
    },
    editOk(row) {
      classStudentSubjectScoreUpdateScore({
        id: row.id,
        score: row.score
      }).then(res => {
        if (res.data.code == 200) {
          this.$message.success('修改成功')
          row.hoverShow = false
          this.reload()
        } else {
          this.$message.error(res.data.message)
        }
      })
      //   if (row.hoverShow) {
      //     row.hoverShow = false
      //   } else {
      //     row.hoverShow = true
      //   }
    },
    LookMeHistory(row) {
      this.$router.push({
        name: 'StudentMarkListhistory',
        query: {
          studentName: row.studentName,
          term: row.term,
          classId: row.classId,
          studentId: row.studentId,
          subjectId: row.subjectId
        }
      })
    },

    CheckChange(val, item,index) {
      // console.log(val,item,index);
      if(val){
          if(index==1){
              item.classRanking = 1
          }else{
            item.averageScore = 1
          }
      }else{
        if(index==1){
              item.classRanking = 0
          }else{
            item.averageScore = 0
          }
      }
      
      // if (val) {
      //   if (item.averageScore) {
      //     item.averageScore = 1
      //   } else if (item.classRanking) {
      //     item.classRanking = 1
      //   }
      // } else {
      //   if (item.averageScore) {
      //     item.averageScore = 0
      //   } else if (item.classRanking) {
      //     item.classRanking = 0
      //   }
      // }
      // console.log(val, item)
    },
    sendMark(event,time) {
      if(this.sendScores.some((item,index,array)=>{return item.flag})){
      let sendScores = []
      let obj = {}
      obj.subjectId = this.queryObj.subjectId
      this.sendScores.forEach((item, index) => {
        if (index == 0) {
          obj.averageScore = item.averageScore
        } else if (index == 1) {
          obj.classRanking = item.classRanking
        }
      })
      obj.gradeRanking = 0
      sendScores.push(obj)
      let object = {
        classId: this.queryObj.classId,
        term: this.queryObj.term,
        examType: this.queryObj.examType,
        examName:this.queryObj.examName,
        type: 2,
        teahcerType: 0,
        gradeId: this.queryObj.gradeId,
        sendScores: sendScores,
        sendTime:time?time:null
      }
        if(!time){
          classStudentSubjectScoreSendScore(object).then(res => {
            if (res.data.code == 200) {
              this.$message({
                type: 'success',
                message: res.data.message
              })
              this.dialogVisible2 = false
            } else {
              console.log(res)
            }
          })
        }else{
          if(new Date(time).getTime()>Date.now()){
          classStudentSubjectScoreSendScoreByQuartzTime(object).then(res=>{
              if(res.data.code==200){              
                this.$message({
                  type: 'success',
                  message: res.data.message
                })
                this.dialogVisible2 = false;
                this.dialogVisible2_1=false
              }else{
                console.log(res);
              }
            })
             }else{  
           this.$message({
              type: 'warning',
              message: '您设置的定时时间已经过期'
            })
          }
          
        }

      }else{
         this.$message({
            type: 'warning',
            message: '至少选择一个'
        })
      }
    }
  },
  computed: {},
  created() {
    console.log('this.queryObj:', this.queryObj)
    classStudentSubjectScoreClassStudentSubjectScoreList({
      classId: this.queryObj.classId,
      type: 2,
      term: this.queryObj.term,
      examType: this.queryObj.examType,
      subjectId: this.queryObj.subjectId,
      examName:this.queryObj.examName
    }).then(res => {
      if (res.data.code == 200) {
        console.log(res.data.data)
        res.data.data.data.forEach(item => {
          item.hoverShow = false
        })
        this.tableData = res.data.data
      } else {
        console.log(res)
      }
    })
  }
}
</script>

<style lang="scss">
</style>
