<template>
  <div id="survey">
    <div v-show="!isLoaded" class="loadingbg">
      <!-- <div class="loading"> </div> -->
    </div>
    <div v-show="isLoaded">
      <div class="tmp"></div>
      <div id= "background"> <img src="../assets/wallpaper.jpg" @load="loaded">
      <img src="../assets/ball.gif" @load="loaded">
      <img src="../assets/lin.png" @load="loaded">
      <img src="../assets/bruh.png" @load="loaded"></div>
    <div v-show="introstage" class="intro">
      <img class="wifi" src="../assets/wifi2.png" @load="loaded"/>
      <img class="battery" src="../assets/battery2.png" @load="loaded"/>
      <img class="signal" src="../assets/signal2.png" @load="loaded"/>
      <div class="carrier">中国移动</div>
      <div class="time">{{timenow}}</div>
      <div class="date">{{datenow}}</div>
      <br />
      <button class="startbutton" @click="startQuiz" >
        <div class="tophalf">
          <img class="wechat" src="../assets/wechat.png" @load="loaded"/>
          <span class="btntxt2">微信</span>
          <span class="btntxt3">现在</span>
        </div>
        <span class="btntxt4">林医师👨‍⚕️</span>
        <span class="btntxt">你有一条来自问止中医首席医疗官的消息</span>
      </button>
      <div class="shimmer" @click="startQuiz">点击解锁</div>
    </div>
    <div v-show="questionstage" class="stage2">
      {{avatar}}
      <div class="time2">{{timenow}}</div>
      <img class="wifi" src="../assets/wifi.png" @load="loaded"/>
      <img class="battery" src="../assets/battery.png" @load="loaded"/>
      <img class="signal" src="../assets/signal.png" @load="loaded"/>
      <div class="time2">{{timenow}}</div>
      <img class="prev" src="../assets/lessthan.png" @click="refresh" @load="loaded"/>
      <div class="title">林医师👨‍⚕️</div>
      <img class="dot" src="../assets/dot.png" @load="loaded"/>
      <div class="container" id="container">
        <div class="prologue">
          {{datetime}}
          <br />You have added 林医师 as your WeChat contact. <br>Start Chatting!
        </div>
        <img class="gif" src="../assets/aaa.gif" @load="loaded"/>
        <img src="../assets/lin.png" class="fpic">
        <div v-for="(msg, index) in messages" class="message"  v-bind:key="msg+index" :class="{'message-out': msg.person ==='you', 'message-in': msg.person === 'doc', 'message-a': msg.person==='a'}">
          <div v-show="msg.person==='doc'">
            <img src="../assets/lin.png" class="pic" />
            <div class="triangle-left"></div>
          </div>
          <div v-show="msg.person==='you'">
            <div v-if="avatar === ''">
              <img src="../assets/bruh.png" class="pic2" />
            </div>
            <div v-else>
              <img id="pic" class="pic2"/>
            </div>
            <div class="triangle-right"></div>
          </div>
            <div class="msg">{{msg.body}}</div>
        </div>
      </div>
      <div class="cbar">
        <img class="talk" src="../assets/talk.png" @load="loaded"/>
        <div class="blank"></div>
        <img class="smile" src="../assets/smile.png" @load="loaded"/>
        <img class="plus" src="../assets/plus.png" @load="loaded"/>
      </div>
      <div class="enter">{{entermsg}}</div>
      <div class="keyboard">
        <form class="form" name="form_name" id="form_name">
          <table align="center" id="table1">
            <tr class="clickable" id="opt" @click="option1">
              <td>
                <input type="button" class="a" id="a" value=""/>
              </td>
            </tr>
            <tr class="clickable" id="opt" @click="option2">
              <td>
                <input type="button" class="a" id="b" value=""/>
              </td>
            </tr>
          </table>
        </form>
      </div>
    </div>
    <div v-show="resultstage" class="resultstage">
      <div class="title2">测试结果</div>
      <div class="text2">您好，你的测试结果如下，也可以直接打印查看哦～</div>
      <div class="flex">
        <button class="printButton" @click="print">打印</button>
        <button class="refreshButton" @click="refresh">返回首页</button>
      </div>
      <canvas id="myChart" class="chart"></canvas>
      <br />
    </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import axios from "axios";
import Chart from "chart.js";
import { setTimeout } from "timers";
// import JQuery from 'jquery';
export default {
  name: "survey",
  data() {
    return {
      diag: "",
      entermsg: "问止输入法",
      count: 0,
      isLoaded: false,
      messages: [
        {body: '', person:"a"},
        {body: '', person:"a"},
        {body: '', person:"a"},
        {body: '', person:"a"},
        {body: '', person:"a"},
       ],
      timenow: "",
      datenow: "",
      datetime: "",
      arr: { //match P0ST format
        answers: []
      },
      introstage: true,
      questionstage: false,
      resultstage: false,
      results: "",
      avatar:'',
    };
  },
  methods: {
    time() {
      this.timenow = moment().format("h:mm");
      this.datenow = moment().format("dddd, MMMM D");
    },
    option1() {
      this.arr.answers.push({id:this.results.data.next.id, optionValue:this.results.data.next.options[1].value});
      this.messages.push({
        body: this.results.data.next.options[1].text,
        person: "you"});
      this.results="";
      document.getElementById('a').disabled = true;
      document.getElementById('b').disabled = true;
      this.entermsg = "对方正在输入...";
      this.nextQuestion();
    },
    option2() {
     this.arr.answers.push({id:this.results.data.next.id, optionValue:this.results.data.next.options[0].value});
      this.messages.push({
        body: this.results.data.next.options[0].text,
        person: "you"});
      this.results="";
      document.getElementById('a').disabled = true;
      document.getElementById('b').disabled = true;
      this.entermsg = "对方正在输入...";
      this.nextQuestion();
    },
    print() {
      window.print();
    },
    async submitForm() {
      var elem = document.getElementById("container");
      var c = this;
      var len = this.results.data.result.physiqueResults.length;
      var max = -1;
      var index = -1;
      for(var i =0; i < len; i++){
        if (this.results.data.result.physiqueResults[i].score > max){
          max = this.results.data.result.physiqueResults[i].score;
          index = i;
        }
      }
      this.messages.push({
        body:this.results.data.result.physiqueResults[index].name + "体质",
        person:"doc"
      }); 

      // const url = "/acup/dialectical/physique/{physiqueId}/conditioning";
      // await axios.get(url, {params: {physiqueId: index}}).then(response => (this.diag = response.data));
      // alert(this.diag.data.physiqueName);
      
      setTimeout(function(){elem.scrollTop = elem.scrollHeight;},0);
      setTimeout(function(){
      c.questionstage = false;
      c.resultstage = true;
      c.drawChart();
      },2500);

    },
    startQuiz() {
      var c = this;
      c.entermsg = "对方正在输入...";
      setTimeout(function(){
      c .messages.push({body:"接下来，为了了解你的健康状况，麻烦你简要回答一下如下问题:", person:"doc"});
      c.entermsg = "问止输入法"
      }, 1500);
      setTimeout(function(){
        c.entermsg = "对方正在输入...";
      },2000);
      setTimeout(function(){
      const url = "/acup/dialectical/physique/question/next";
      axios.post(url, c.arr).then(response => (
      (c.results = response.data),
      c.messages.push({body: c.results.data.next.question + "?",person: "doc"}),
      document.getElementById('a').value=c.results.data.next.options[1].text,
      document.getElementById('b').value=c.results.data.next.options[0].text,
      c.entermsg = "问止输入法"
      ));
      },3000);
      c.datetime = moment().format("MMM D, YYYY h:mm A");
      c.introstage = false;
      c.questionstage = true;

    },
    async nextQuestion() {
      var elem = document.getElementById("container");
      setTimeout(function() {
          elem.scrollTop = elem.scrollHeight;
      }, 0);
      //try to get a new question
      const url = "/acup/dialectical/physique/question/next";
      await axios.post(url, this.arr).then(response => (
      this.results = response.data));

      // if question exists, display & return
      if(this.results.data.next != null) {
        var c = this;
        setTimeout(function() {
          c.messages.push({
            body: c.results.data.next.question + "?",
            person: "doc"});
            elem.scrollTop = elem.scrollHeight;
        }, 1250);
        setTimeout(function() {
          elem.scrollTop = elem.scrollHeight;
          var bt = document.getElementById("a");
          var ct = document.getElementById("b");
          bt.value = c.results.data.next.options[1].text;
          ct.value = c.results.data.next.options[0].text;
          document.getElementById('a').disabled = false;
          document.getElementById('b').disabled = false;
          c.entermsg = "问止输入法";
        }, 1250);
        return;
      }
      //if finished, submit 
      if (this.results.data.result != null) {
        var d = this;
        setTimeout(function(){d.messages.push({body: "有结果了！", person: "doc"})},500);
        setTimeout(function(){elem.scrollTop = elem.scrollHeight},500);
        setTimeout(function(){d.submitForm(); return;}, 2500);
      }
    },
 
    drawChart() {
      Chart.defaults.global.defaultFontFamily = "FZQKBYSJW--GB1-0";
      var sp_a = this.results.data.result.physiqueResults[0].score;
      var sp_b = this.results.data.result.physiqueResults[1].score;
      var sp_c = this.results.data.result.physiqueResults[2].score;
      var sp_d = this.results.data.result.physiqueResults[3].score;
      var sp_e = this.results.data.result.physiqueResults[4].score;
      var sp_f = this.results.data.result.physiqueResults[5].score;
      var sp_g = this.results.data.result.physiqueResults[6].score;
      var sp_h = this.results.data.result.physiqueResults[7].score;
      var ctx = document.getElementById("myChart");
      // eslint-disable-next-line
      var myChart = new Chart(ctx, {
        type: "bar",
        data: {
          labels: [
            "脾阳虚",
            "肾阳虚",
            "血虚",
            "阴虚",
            "气滞",
            "湿热",
            "血瘀",
            "痰湿"
          ],
          datasets: [
            {
              backgroundColor: [
                "rgba(102, 255, 102, 0.3)",
                "rgba(102, 204, 255, 0.3)",
                "rgba(255, 206, 86, 0.2)",
                "rgba(255, 99, 132, 0.2)",
                "rgba(153, 102, 255, 0.2)",
                "rgba(51, 153, 51, 0.2)",
                "rgba(255, 102, 0, 0.2)",
                "rgba(0, 153, 255, 0.2)"
              ],
              borderColor: [
                "rgba(102, 255, 102, 1)",
                "rgba(102, 204, 255, 1)",
                "rgba(255, 206, 86, 1)",
                "rgba(255, 99, 132, 1)",
                "rgba(153, 102, 255, 1)",
                "rgba(51, 153, 51, 1)",
                "rgba(255, 102, 0, 1)",
                "rgba(0, 153, 255, 1)"
              ],
              borderWidth: 1,
              label: "体质分析量化图",
              data: [sp_a, sp_b, sp_c, sp_d, sp_e, sp_f, sp_g, sp_h]
            }
          ]
        },
        options: {
          scales: {
            xAxes: [{ ticks: { beginAtZero: true, fontSize: 22 } }],
            yAxes: [{ ticks: { beginAtZero: true, fontSize: 22 } }]
          },
          legend: {
            labels: {
              fontSize: 22
            }
          },
          responsive: true,
          maintainAspectRatio: false
        }
      });
    }, //end drawChart() function
    refresh() {
      window.location.reload();
    },
    loaded(){
      this.count++;
      if(this.count == 17)
      this.isLoaded = true;
    }
  },
  created(){
    // const url = "/wechat/service/login";
    var currUrl= window.location.href;
    var a = currUrl.indexOf('=') + 1;
    var code = currUrl.substr(a,);
    // alert(code);
    // axios.get(url,{params:{redirect_url: 'http://192.168.0.128:8080/login'}}).then(response => (code = response.data));
    const url =  "/wechat/service/userinfo";
    axios.get(url,{params:{code: code,}}).then(response => (this.avatar = response.data.avatarUrl));
    var img = new Image;
    img.src = this.avatar;
    document.getElementById('pic').src = img.src;
  },
  mounted() {
    this.interval = setInterval(this.time, 1000);
  }
};
</script>

<style lang="scss">
@keyframes temp{
  0%{
    opacity:1
  }
  100%{
    opacity: 0;
    z-index:-1;
  }
}
.tmp::after{
  background-image:url(../assets/ball.gif);
  background-size:cover;
  position:absolute;
  top:0px;
  left:0px;
  width:100%;
  height:100%;
  content:"";
  z-index:1;
  animation:temp 5s;
  animation-fill-mode:forwards;

}

.triangle-left{
  width:0;
  height:0;
  border-top:2px solid transparent;
  border-right:4px solid white;
  border-bottom:2px solid transparent;
  position:absolute;
  margin-left:-12px;
  margin-top:2px;

}
.triangle-right{
  width:0;
  height:0;
  border-top:2px solid transparent;
  border-left:4px solid rgb(82, 226, 53);
  border-bottom:2px solid transparent;
  position:absolute;
  right:56px;
  margin-top:2px;
}
#background{
  display:none;
}
.loadingbg::after{
  background:#0e0e0e;
  background-size:cover;
  content:"";
  top:0px;
  left:0px;
  position:absolute;
  z-index:-1;
  width:100%;
  height:100%;
}
.loading{
  position:absolute;
  top: 30%;
  left: 50%;
  margin-left:-36px;
  border: 10px solid #f3f3f3; 
  border-top: 10px solid rgb(82, 226, 53);
  border-radius:50%;
  width: 30px;
  height: 30px;
  animation: spin 1s linear infinite;
}
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
.gif{
  -webkit-appearance:none;
  position:absolute;
  left:60px;
  height:155px;
  width:100px;
}
.enter{
  padding-top:5px;
  background:rgb(221, 221, 221);
  position:relative;
  font-size:14px;
  top:50px;
}
.resultstage{
  animation: slide2 1.5s;
}
@keyframes slide2{
  from {
     transform:rotate(0deg) scale(0);
  }
  to{
     transform:rotate(1440deg)  scale(1);
  }
}
.a {
  appearance:none; 
  -moz-appearance: none;
  -webkit-appearance: none; 
  border-radius: 10px 10px 10px 10px;
  background: rgb(255, 255, 255);
  margin-right: 10px;
  margin-left: 10px;
  font-size: 50px;
  height: 120px;
  width: 120px;
}
.a:focus {
  outline: 0;
}
.plus {
  position: absolute;
  top: 8px;
  right: 5%;
  height: 27px;
  width: 27px;
}
.smile {
  position: absolute;
  top: 8px;
  right: 15%;
  height: 34px;
  width: 37px;
}
.talk {
  position: absolute;
  top: 6px;
  left: 4%;
  height: 30px;
  width: 30px;
}
.blank {
  position: absolute;
  left: 15%;
  top: 4px;
  background-color: white;
  border-radius: 5px 5px 5px 5px;
  height: 34px;
  width: 60vw;
}
table tr {
  display: table-cell;
}
.keyboard {
  position: relative;
  top: 50px;
  height: 19.2vh;
  width: 100vw;
  background: rgb(221, 221, 221);
}
.cbar {
  background: rgb(244, 244, 244);
  position: relative;
  height: 44px;
  width: 100vw;
  top: 50px;
}
.dot {
  position: absolute;
  top: 50px;
  right: 4%;
  height: 25px;
  width: 25px;
}
.prev {
  cursor: pointer;
  z-index: 99;
  position: absolute;
  top: 52px;
  left: 5%;
  height: 20px;
  width: 10px;
}
.stage2 {
  animation: slide 1s;
  position: absolute;
  top: 0px;
  height: 100%;
  width: 100%;
  left: 0px;
  background: rgb(240, 240, 240);
}
@keyframes slide {
  from {
    transform: scale(3.0)
  }
  to {
    transform: scale(1.0)
  }
}

@keyframes fadein {
  from {
    transform: scale(0);
  }
  60% {
    transform: scale(0);
  }
  to {
    transform: scale(1);
  }
}
.shimmer {
  cursor:pointer;
  position: absolute;
  top: 85%;
  left: 50%;
  margin-left: -50px;
  font-size: 25px;
  background: #868585 -webkit-gradient(linear, 100% 0, 0 0, from(#acacac), color-stop(0.5, #ffffff), to(#acacac));
  background-position: -4rem top; /*50px*/
  background-repeat: no-repeat;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation-name: shimmer;
  animation-duration: 1.4s;
  animation-iteration-count: infinite;
  background-size: 4rem 100%; /*50px*/
}

@keyframes shimmer {
  0% {
    background-position: -4rem top; /*50px*/
  }
  100% {
    background-position: 8rem top; /*200px*/
  }
}

.carrier {
  position: absolute;
  top: 5px;
  font-size: 14px;
  font-weight: bold;
  color: white;
}
.prologue {
  font-size: 12px;
  color: #666;
  padding-bottom: 1em;
}
.message {
  max-width: 60%;
  border-radius: 10px;
  padding: 0.5em;
  font-size: 0.8em;
  margin: 0 auto 0.3em auto;
}
.fpic{
  height:35px;
  width:35px;
  border-radius: 5px 5px 5px 5px;
  position:absolute;
  left:17px;
}
.pic {
  margin-top: -7px;
  height: 35px;
  width: 35px;
  position: absolute;
  border-radius: 5px 5px 5px 5px;
  margin-left:-50px;
}
.pic2 {
  margin-top: -7px;
  height: 37px;
  width: 37px;
  position: absolute;
  border-radius: 5px 5px 5px 5px;
  right:17px;

}
@keyframes leftfade{
  0%{
    opacity:0;
    transform:translateX(-50px);
  }
  100%{
    transform:translateX(0px);
    opacity:1;
  }
}
@keyframes rightfade{
  0%{
    opacity:0;
  }
  100%{
    opacity:1;
  }
}
.message-in {
  width:fit-content;
  animation-name: leftfade;
  animation-duration: 1s;
  background: white;
  color: rgb(0, 0, 0);
  margin-left: 40px;
  text-align: left;
}
.message-out {
  width:fit-content;
  animation-name: rightfade;
  animation-duration: 1s;
  background: rgb(82, 226, 53);
  color: black;
  margin-right: 40px;
}
.message-a {
  height:15.5px;
  color:rgb(240, 240, 240);
  margin-bottom:0px;
}
.container {
  padding: 1em;
  position: relative;
  top: 50px;
  overflow: auto;
  height: 56vh;
}
.time2 {
  position: absolute;
  top: 10px;
  left: 20px;
  font-size: 14px;
  font-weight: bold;
}
.signal {
  position: absolute;
  top: 7px;
  right: 54px;
  width: 16px;
  height: 13px;
}
.battery {
  position: absolute;
  top: 5px;
  right: 10px;
  width: 20px;
  height: 20px;
}
.wifi {
  position: absolute;
  top: 5px;
  right: 32px;
  width: 20px;
  height: 18px;
}
.date {
  color: white;
}
.time {
  color: white;
  padding-top: 50px;
  font-size: 66px;
  font-family: "HelveticaNeue-Light", "Helvetica Neue Light";
}
.clickable {
  cursor: pointer;
}
.chart {
  min-height: 400px;
  max-height: 400px;
  min-width: 700px;
  max-width: 700px;
}
.intro {
  overflow: hidden;
}
.intro::after {
  // background-color:rgb(148, 25, 25);
  background-image: url(../assets/wallpaper.jpg);
  background-repeat: no-repeat;
  background-size: cover;
  content: "";
  position: absolute;
  top: 0;
  left: 0px;
  width: 100%;
  height: 100%;
  opacity: 1;
  z-index: -1;
}
.title {
  position: relative;
  top: 50px;
  font-size: 20px;
  color: #000000;
  text-align: center;
  border-bottom: 0.1px solid rgb(221, 221, 221);
}

.wechat {
  position: absolute;
  left: 0%;
  width: 18px;
  height: 18px;
}
.tophalf {
  position: absolute;
  height: 50%;
  width: 100%;
  top: 15%;
  left: 3%;
}
.btntxt2 {
  position: absolute;
  left: 25px;
  font-size: 12px;
  color: #4a4a4a;
}
.question {
  font-size: 28px;
}
.btntxt3 {
  position: absolute;
  right: 7%;
  font-size: 12px;
  color: #4a4a4a;
}
.background {
  background: #6e7d8a;
}
.btntxt {
  position: absolute;
  bottom: 10%;
  width: 100;
  left: 3%;
  font-size: 14px;
  font-weight: bold;
}
.btntxt4 {
  position: absolute;
  bottom: 33%;
  width: 100;
  left: 3%;
  font-size: 14px;
  font-weight: bold;
}
.startbutton {
  animation: fadein 4s;
  position: relative;
  margin-top: 0px;
  color: black;
  height: 80px;
  width: 95vw;
  background: white;
  opacity: 0.9;
  border-style: none;
  border-radius: 10px 10px 10px 10px;
  transition: all 0.5s;
  cursor: pointer;
}
.text2 {
  font-family: FZQKBYSJW--GB1-0;
  font-size: 30px;
  color: #4a4a4a;
  letter-spacing: 0;
  text-align: center;
  margin-bottom: 75px;
}
.title2 {
  font-family: FZQKBYSJW--GB1-0;
  font-size: 80px;
  color: #4a4a4a;
  letter-spacing: 0;
  text-align: center;
  margin-bottom: 52px;
}
.refreshButton {
  font-family: FZQKBYSJW--GB1-0;
  font-size: 30px;
  color: #ffffff;
  letter-spacing: 0;
  text-align: center;
  background: #6e7d8a;
  border-radius: 10px 10px 10px 10px;
  height: 72px;
  width: 164px;
  cursor: pointer;
  transition: all 0.5s;
}

.refreshButton:hover {
  color: #6e7d8a;
  background-color: white;
  border-style: dashed;
}
.flex {
  position: absolute;
  top: 83px;
  right: 20px;
}
.printButton {
  margin-right: 40px;
  font-family: FZQKBYSJW--GB1-0;
  font-size: 30px;
  color: #ffffff;
  letter-spacing: 0;
  text-align: center;
  background: #891f24;
  border-radius: 10px 10px 10px 10px;
  height: 72px;
  width: 164px;
  transition: all 0.5s;
  cursor: pointer;
}
.printButton:hover {
  background: white;
  color: #891f24;
  border-style: dashed;
}
.printButton:focus {
  outline: 0;
}
canvas {
  margin: 0 auto;
}

@media screen and (min-width: 259px) and (max-width: 823px) {
  .a{
    padding: 6px;
  }
  .refreshButton {
    font-size: 22px;
    width: 40%;
    height: 50%;
  }
  .title2 {
    margin-top:-20px;
    font-size: 30px;
    text-align: center;
    margin-bottom:30px;
  }
  .text2 {
    text-align: center;
    font-size: 24px;
    margin-bottom:30px;
  }
  .printButton {
    margin-right: 5px;
    font-size: 22px;
    width: 40%;
    height: 50%;
  }
  .flex {
    position: relative;
    top: 0px;
    right: 0px;
  }
  .chart {
    max-width: 350px;
    min-width: 350px;
    min-height: 400px;
    max-height: 400px;
    margin-top: 30px;
  }
}
@media print {
  body * {
    visibility: hidden;
  }
  .title2 {
    font-size: 30px;
    text-align: center;
    visibility: visible;
    position: absolute;
    margin-top: -50px;
    margin-left: 40%;
  }
  .chart {
    transform: scaleX(0.5);
    position: absolute;
    margin-top: 75px;
    top: 0px;
    visibility: visible;
  }
}
</style>