<template>
  <el-container>
    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <span>该考试已成功提交，得分{{score}} 请查看成绩详情</span>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="pushExamAnswer()">确 定</el-button>
      </span>
    </el-dialog>
    <el-header>
      <el-form :inline="true" v-model="texts" label-width="300px">
        <el-form-item label="试卷名称：" class="labeiSize">{{texts.pname}}</el-form-item>
        <el-form-item label="学生姓名：" class="labeiSize">{{texts.uid}}</el-form-item>
        <el-form-item label="测验限时：" class="labeiSize">{{texts.ptime}}min</el-form-item>
        <el-form-item label="倒计时：" class="labeiSize">{{keepTime}}</el-form-item>
      </el-form>
    </el-header>
    <el-container>
      <el-aside width="300px">
        <div class="paper-left">
          <div class="paper-title">
            <h1>
              <i class="el-icon-s-grid"></i>答题卡
            </h1>
          </div>
          <el-collapse>
            <el-collapse-item>
              <template slot="title">
                <h2>单选题</h2>
                <span>共{{texts.queNum}}题</span>
              </template>
              <el-button
                class="answer-button"
                circle
                size="small"
                v-show="typeof item.sanswer=='undefined'"
                v-for="(item,index) in exams"
                :key="index+'1'"
                @click.native="jump(index)"
              >{{index+1}}</el-button>
              <el-button
                class="answer-buttong"
                circle
                size="small"
                v-show="typeof item.sanswer!='undefined'"
                v-for="(item,index) in exams"
                :key="index+'2'"
                @click.native="jump(index)"
              >{{index+1}}</el-button>
              <!-- <el-button  class="answer-button" circle size="small" v-for="index of item.count" :id="'answer'+item.code+index"  @click.native="jump(item.code+index)">{{index}}</el-button> -->
            </el-collapse-item>
            <el-collapse-item>
              <template slot="title">
                <h2>填空题</h2>
                <span>共{{texts.blankNum}}题</span>
              </template>
              <el-button
                class="answer-button"
                circle
                size="small"
                v-for="(item,index) in blank"
                :key="index+'3'"
              >{{index+1}}</el-button>
            </el-collapse-item>
          </el-collapse>
        </div>
      </el-aside>
      <el-main>
        <el-card class="box-card" ref="htmlHeight">
          <div slot="header" class="clearfix">
            <span>单选题</span>
          </div>
          <el-card v-for="(item,index) in exams " :key="index+'4'" class="text item" ref="examHeight">
            <div slot="header" class="clearfix">
              <span>{{index+1}}、{{item.content}}（5分）</span>
            </div>
            <el-radio-group v-model="item.sanswer">
              <el-radio label="A">A、{{item.aoption}}</el-radio>
              <br />
              <el-radio label="B">B、{{item.boption}}</el-radio>
              <br />
              <el-radio label="C">C、{{item.coption}}</el-radio>
              <br />
              <el-radio label="D">D、{{item.doption}}</el-radio>
            </el-radio-group>
          </el-card>
        </el-card>
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>填空题</span>
          </div>
          <el-card v-for="(item,index) in blank" :key="index+'5'" class="text item">
            <div slot="header" class="clearfix">
              <span>{{index+1}}、{{item.content}}（5分）</span>
            </div>
            <el-input v-model="item.sanswer"></el-input>
          </el-card>
        </el-card>
        <el-button type="primary" class="summit-button" @click="summmitExam()">提交测验</el-button>
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
// import axios from "axios";
export default {
  created() {
    // 页面被创建时执行一次查询函数
    // this.acceptData();
    this.StartCountDown();
    this.acceptExam();
    this.acceptExamQuestion();
    this.acceptExamBlank();
    //初始化表单
    // this.search = this.searchModel;
    // this.search.maxSurTime = this.getNowFormatDate(0);
    // this.search.minSurTime = this.getNowFormatDate(10);
  },
  data() {
    return {
      dialogVisible: false,
      keepTime: "",
      limittime: 90,
      settime: "",
      flag: false,
      score: "",
      texts: {
        uid: "",
        pid: "",
        startime: "",
        endtime: "",
        time: "",
        ptime: "",
        score: "",
        name: "",
        pname: ""
      },
      exam: [],
      exams: {
        aoption: "",
        boption: "",
        coption: "",
        doption: "",
        answer: "",
        sanswer: ""
      },
      answerstu: null,
      blank: {
        bid: "",
        content: "",
        answer: "",
        pid: ""
      }
    };
  },
  methods: {
    pushExamAnswer() {
      let id = this.$route.params.index;
      const routeUrl = this.$router.resolve({
        path: `/grade/${id}`
      });
      window.open(routeUrl.href);
    },
    acceptExam() {
      let id = this.$route.params.index;
      // 从后端获取考试的详细信息，渲染到考试详情里
      this.$axios
        .get("http://101.200.135.43:8888/paper/getPaper?getPId=" + id, {})
        .then(res => {
          this.texts = res.data;
          this.limittime = res.data.ptime;
          console.log("texts" + this.limittime);
        })
        .catch(err => {
          console.log(err);
        });
    },
    acceptExamQuestion() {
      let id = this.$route.params.index;
      this.$axios
        .get("http://101.200.135.43:8888/paper/getQuestions?getQId=" + id)
        .then(res => {
          this.exams = res.data;
          console.log("exam" + this.exams);
        })
        .catch(err => {
          console.log(err);
        });
    },
    acceptExamBlank() {
      let id = this.$route.params.index;
      this.$axios
        .get("http://101.200.135.43:8888/paper/getBlanks?getBId=" + id)
        .then(res => {
          this.blank = res.data;
          console.log(this.blank);
        })
        .catch(err => {
          console.log(err);
        });
    },
    StartCountDown() {
      var thistime = parseInt(this.texts.ptime);
      console.log("time" + thistime);
      var mydate = new Date();
      mydate.setMinutes(mydate.getMinutes() + this.limittime);
      this.settime = mydate;
      let time = setInterval(() => {
        if (this.flag == true) {
          clearInterval(time);
        }
        this.timeDown();
      }, 100);
    },
    timeDown() {
      const endTime = new Date(this.settime);
      const nowTime = new Date();
      let leftTime = parseInt((endTime.getTime() - nowTime.getTime()) / 1000);
      // let d = parseInt(leftTime / (24 * 60 * 60));
      let h = this.formate(parseInt((leftTime / (60 * 60)) % 24));
      let m = this.formate(parseInt((leftTime / 60) % 60));
      let s = this.formate(parseInt(leftTime % 60));
      if (leftTime <= 0) {
        this.flag = true;
        // alert("时间到，停止作答");
      }
      this.keepTime = `${h}:${m}:${s}`;
    },
    formate(time) {
      if (time >= 10) {
        return time;
      } else {
        return `0${time}`;
      }
    },
    summmitExam() {
      let len = this.exams.length;
      let lens = this.blank.length;
      let l = len + lens;
      let i, j;
      let id = this.$route.params.index;
      this.answerstu = this.exams;
      // this.answerstu.length = l;
      for (i = 0; i < len; i++) {
        delete this.answerstu[i].aoption;
        delete this.answerstu[i].boption;
        delete this.answerstu[i].coption;
        delete this.answerstu[i].doption;
        delete this.answerstu[i].answer;
        delete this.answerstu[i].content;
        this.$set(this.answerstu[i], "teid", null);
        this.$set(this.answerstu[i], "quid", this.answerstu[i].qid);
        delete this.answerstu[i].qid;
        this.$set(this.answerstu[i], "anid", null);
        this.$set(this.answerstu[i], "uid", 1);
        this.$set(this.answerstu[i], "statement", null);
        this.$set(this.answerstu[i], "pid", id);
        this.$set(this.answerstu[i], "anid", null);
        this.$set(this.answerstu[i], "sanswer", this.exams[i].sanswer);
        // this.answerstu[i].sanswer=this.exams[i].sanswer;
      }
      for (j = len, i = 0; j < l, i < lens; j++, i++) {
        this.answerstu[j] = this.blank[i];
        this.$set(this.answerstu[j], "quid", this.blank[i].bid);
        delete this.answerstu[j].bid;
        delete this.answerstu[j].answer;
        delete this.answerstu[j].content;
        delete this.answerstu[j].pid;
        this.$set(this.answerstu[j], "teid", null);
        this.$set(this.answerstu[j], "anid", null);
        this.$set(this.answerstu[j], "uid", 1);
        this.$set(this.answerstu[j], "statement", null);
        this.$set(this.answerstu[j], "pid", id);
        this.$set(this.answerstu[j], "anid", null);
        // this.answerstu[j].quid=this.blank[i].bid;
      }
      let answerList = this.answerstu;
      // let a=JSON.stringify(answerList);
      console.log("s" + answerList);
      this.$axios
        .post("http://101.200.135.43:8888/paper/getScore", {
          data: answerList
        })
        .then(res => {
          this.score = res.data;
        })
        .catch(err => {
          console.log(err);
        });
      this.dialogVisible = true;
      // delete(this.answerstu,'aoption');
      // console.log("answerstu" +a);
    },
    jump(index) {
      // let jump = this.$refs.exams("#" + postion);
      // // 获取需要滚动的距离
      // // let total = jump[0].offsetTop;
      // //实现form锚点定位
      // this.$refs.exams.scrollTop = jump[0].offsetTop;
      let h = this.$refs.examHeight[index].$el.offsetTop;
      this.$refs.htmlHeight.$el.scrollTop = h;
      console.log(this.$refs.htmlHeight.$el.scrollTop);
    }
  }
};
</script>
<style scoped>
.el-radio__input.is-checked .el-radio__inner {
  border-color: red;
  background: red;
}
.month {
  margin-left: 8px;
}
.el-header {
  width: 100%;
  height: 70px;
  font-size: 22px;
  color: #fff;
  background-color: #242f42;
}
.labeiSize > :first-child {
  font-size: 19px;
  color: #fff;
}
.paper-left {
  position: absolute;
  padding: 10px;
  left: 0;
  top: 60px;
  bottom: 0;
  width: 300px;
  overflow-x: hidden;
  overflow-y: auto;
  border: 1px solid #e4e4e4;
  border-top: none;
}
.paper-title {
  width: 98%;
  height: 45px;
  line-height: 45px;
  background: #f7f7f7;
}

.paper-title h1 {
  font-size: 20px;
  margin: 0;
}
.answer-button {
  padding: 0px;
  color: #0a0a0a;
  background-color: #ffffff;
  border-color: #e4e4e4;
  margin-left: 10px;
  width: 30px;
  height: 30px;
}
.answer-button:hover {
  background: #ecf1ef;
  border-color: #e4e4e4;
  color: #0a0a0a;
}
.answer-button-check {
  background: #13ce66;
  border-color: #30b08f;
}
.answer-buttong {
  background: #13ce66;
  padding: 0px;
  color: #0a0a0a;
  border-color: #30b08f;
  margin-left: 10px;
  width: 30px;
  height: 30px;
}
.answer-radio {
  display: list-item;
  margin: 5px 0px;
}

.el-collapse-item h2 {
  width: 150px;
  font-size: 14px;
  display: inline-block;
}
.el-form--label-top >>> .el-form-item__label {
  float: none;
  display: inline-block;
  text-align: left;
  padding: 0px;
}

.el-card {
  margin: 10px;
}

.el-card >>> .el-card__header {
  background-color: #ffffff;
  padding: 0px 10px;
  line-height: 35px;
  font-size: 16px;
}
.el-card >>> .el-card__body {
  padding: 5px 20px;
}
.summit-button {
  position: fixed;
  right: 5%;
  top: 80%;
}
</style>