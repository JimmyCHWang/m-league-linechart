<template>
  <div class="container py-3" id="form">
    <div id="switches-and-buttons">
      <button type="button" class="btn btn-warning" id="fix-names" @click="openModal()">修改姓名</button>
      <!-- Change Names Button / 名前を変更 -->
    </div>
    <div id="normal-block">
      <div class="text-center mb-3">
        <h2>{{getRound(currentPlayInfo, true)}}</h2>
      </div>
      <form>
        <div class="mb-2">
          <div class="input-group mb-3">
            <span class="input-group-text" id="east-name">{{ playerNames.east }}</span>
            <input type="text" class="form-control" :placeholder="normalFormInfo.eastLastScore" v-model="normalFormInfo.eastScore">
          </div>
          <div class="input-group mb-3">
            <span class="input-group-text" id="south-name">{{ playerNames.south }}</span>
            <input type="text" class="form-control" :placeholder="normalFormInfo.southLastScore" v-model="normalFormInfo.southScore">
          </div>
          <div class="input-group mb-3">
            <span class="input-group-text" id="west-name">{{ playerNames.west }}</span>
            <input type="text" class="form-control" :placeholder="normalFormInfo.westLastScore" v-model="normalFormInfo.westScore">
          </div>
          <div class="input-group mb-3">
            <span class="input-group-text" id="north-name">{{ playerNames.north }}</span>
            <input type="text" class="form-control" :placeholder="normalFormInfo.northLastScore" v-model="normalFormInfo.northScore">
          </div>
          <div class="form-check fs-4 ms-4">
            <label class="form-check-label" for="normalCounterCheckbox">连庄</label>
            <!-- Dealer remains the same / 連荘-->
            <input class="form-check-input" type="checkbox" value="" id="normalCounterCheckbox" v-model="normalFormInfo.addCounter">
          </div>
        </div>
        <div class="text-center">
          <button type="button" class="btn btn-primary me-3" @click="addRecordNormal()">添加记录</button>
          <button type="button" class="btn btn-danger" @click="revertRecordNormal()">退回上条</button>
          <!-- Add a record and revert a record / 対局記録を追加または撤回 -->
        </div>
      </form>
    </div>
  </div>
  <div class="container" id="chart">
    <canvas id="normalChart"></canvas>
  </div>
  <div class="modal fade" id="editNameModal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="editNameModalLabel">修改选手名称</h5>
          <!-- Change name modal / 名前変更モーデル -->
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          <input type="text" class="form-control mb-3" placeholder="起始东家" v-model="modalTempNames.east">
          <input type="text" class="form-control mb-3" placeholder="起始南家" v-model="modalTempNames.south">
          <input type="text" class="form-control mb-3" placeholder="起始西家" v-model="modalTempNames.west">
          <input type="text" class="form-control" placeholder="起始北家" v-model="modalTempNames.north">
          <!-- Names of those who sit on E,S,W,N position / 東、南、西、北、四家の名前を入力 -->
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">取消</button>
          <button type="button" class="btn btn-primary" data-bs-dismiss="modal" @click="editNames()">保存</button>
          <!-- Cancel & Save / キャンセルとセーブ -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import {Modal} from 'bootstrap';
import Chart from 'chart.js/auto';

export default {
  name: 'App',
  data() {
    return {
      currentPlayInfo: {
        round: 1,
        hand: 1,
        counter: 0,
        riichiBets: 0,
      },
      playerNames: {
        east: "P1",
        south: "P2",
        west: "P3",
        north: "P4"
      },
      normalFormInfo: {
        eastLastScore: 250,
        southLastScore: 250,
        westLastScore: 250,
        northLastScore: 250,
        eastScore: "",
        southScore: "",
        westScore: "",
        northScore: "",
        addCounter: false,
      },
      modalTempNames: {
        east: "P1",
        south: "P2",
        west: "P3",
        north: "P4"
      },
      editNameModalInstance: undefined,
      normalChartInstance: undefined,
      normalScores: [{
        round: 0,
        hand: 0,
        counter: 0,
        east: 25000,
        south: 25000,
        west: 25000,
        north: 25000,
      }],
    }
  },
  computed : {
    dealer(){return ["east", "south", "west", "north"][this.currentPlayInfo.hand]},
    normalMax() {
      return Math.max(...(this.normalScores.map(o => Math.max(o.east, o.south, o.west, o.north))));
    },
    normalMin() {
      return Math.min(...(this.normalScores.map(o => Math.min(o.east, o.south, o.west, o.north))));
    }
  },
  mounted() {
    this.editNameModalInstance = new Modal(document.getElementById("editNameModal"));
    Chart.defaults.color = 'white';
    Chart.defaults.borderColor = 'white';
    this.renderChart();
    //console.log(calcPoints(3, 40, true, true))
  },
  methods: {
    getRound: function(playInfo, shorten = false) {
      let {round, hand, counter} = playInfo;
      let roundText = {1:"東", 2:"南", 3:"西", 4:"北", 5: "延长"}[round] || "";
      if (shorten) {
        return `${roundText}${hand}局`;
      }
      return `${roundText}${hand}局${counter > 0 ? counter+"本场" : ""}`;
    },
    addRecordNormal() {
      //console.log("add");
      let currInfo = this.currentPlayInfo;
      let formInfo = this.normalFormInfo;
      let timesHundred = function (val, last) {
        if (Number.isNaN(val)) return last * 100;
        if (Math.abs(val) < 1000) return val * 100;
        return val;
      }
      this.normalScores.push({
        round: currInfo.round,
        hand: currInfo.hand,
        counter: currInfo.counter,
        east: timesHundred(Number.parseInt(formInfo.eastScore),formInfo.eastLastScore),
        south: timesHundred(Number.parseInt(formInfo.southScore),formInfo.southLastScore),
        west: timesHundred(Number.parseInt(formInfo.westScore),formInfo.westLastScore),
        north: timesHundred(Number.parseInt(formInfo.northScore),formInfo.northLastScore),
      });
      if (formInfo.addCounter) {
        this.currentPlayInfo.counter++;
      } else {
        this.currentPlayInfo.hand++;
        this.currentPlayInfo.counter = 0;
        if (this.currentPlayInfo.hand > 4) {
          this.currentPlayInfo.round++;
          this.currentPlayInfo.hand = 1;
        }
      }
      this.normalFormInfo = {
        eastLastScore: timesHundred(Number.parseInt(formInfo.eastScore),formInfo.eastLastScore) / 100,
        southLastScore: timesHundred(Number.parseInt(formInfo.southScore),formInfo.southLastScore) / 100,
        westLastScore:  timesHundred(Number.parseInt(formInfo.westScore),formInfo.westLastScore) / 100,
        northLastScore: timesHundred(Number.parseInt(formInfo.northScore),formInfo.northLastScore) / 100,
        eastScore: "",
        southScore: "",
        westScore: "",
        northScore: "",
        addCounter: false,
      };
      //console.log(this.normalScores);
      this.renderChart();
    },
    revertRecordNormal() {
      if (this.normalScores.length <= 1) return;
      this.currentPlayInfo.counter = this.normalScores[this.normalScores.length - 1].counter;
      this.currentPlayInfo.hand = this.normalScores[this.normalScores.length - 1].hand;
      this.currentPlayInfo.round = this.normalScores[this.normalScores.length - 1].round;
      this.normalScores.splice(this.normalScores.length - 1, 1);
      let len = this.normalScores.length;
      this.normalFormInfo = {
        eastLastScore: this.normalScores[len - 1].east,
        southLastScore: this.normalScores[len - 1].south,
        westLastScore:  this.normalScores[len - 1].west,
        northLastScore:  this.normalScores[len - 1].north,
        eastScore: null,
        southScore: null,
        westScore: null,
        northScore: null,
        addCounter: false,
      };
      //console.log(this.normalFormInfo);
      this.renderChart();
    },
    editNames: function () {
      this.playerNames.east = this.modalTempNames.east;
      this.playerNames.south = this.modalTempNames.south;
      this.playerNames.west = this.modalTempNames.west;
      this.playerNames.north = this.modalTempNames.north;
      this.renderChart();
    },
    openModal() {
      this.modalTempNames.east = this.playerNames.east;
      this.modalTempNames.south = this.playerNames.south;
      this.modalTempNames.west = this.playerNames.west;
      this.modalTempNames.north = this.playerNames.north;
      this.editNameModalInstance.show();
    },
    renderChart() {
      let _this = this;
      if (this.normalChartInstance) {
        this.normalChartInstance.stop();
        this.normalChartInstance.destroy();
      }
      this.normalChartInstance = new Chart(document.getElementById("normalChart").getContext('2d'), {
        type: 'line',
        data: {
          labels: this.normalScores.map(o => {
            if (o.round == 0 || o.counter > 0) return "";
            if (o.hand == 1) {return `${{1:"東", 2:"南", 3:"西", 4:"北"}[o.round] || "延"}1`}
            return o.hand;
          }),
          datasets: [{
            label: this.playerNames.east,
            data: this.normalScores.map(o => o.east),
            borderColor: 'red',
            backgroundColor: 'red',
          },{
            label: this.playerNames.south,
            data: this.normalScores.map(o => o.south),
            borderColor: 'blue',
            backgroundColor: 'blue',
          },{
            label: this.playerNames.west,
            data: this.normalScores.map(o => o.west),
            borderColor: 'green',
            backgroundColor: 'green',
          },{
            label: this.playerNames.north,
            data: this.normalScores.map(o => o.north),
            borderColor: 'yellow',
            backgroundColor: 'yellow',
          }]
        },
        options: {
          animation: false,
          scales: {
            x: {
              borderWidth: 3,
              autoSkip: false,
            },
            y: {
              grid: {
                display: false
              },
              ticks: {
                callback: function (val) {
                  return (val == 25000 || val == _this.normalMax || val == _this.normalMin) ? this.getLabelForValue(val) : '';
                },
                stepSize: 100,
                autoSkip: false,
              },
              borderWidth: 3,
              max: this.normalMax+500,
              min: this.normalMin-500,
            }
          }
        }
      })
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

#fix-names {
  position: absolute;
  top: 1rem;
  right: 1rem;
}

canvas {
  background: #111111;
}
</style>
