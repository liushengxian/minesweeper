<template>
  <div>
    <div class="task">
      <div class="smile" :class="{'game-over': gameOver, 'win': win}" @click="restart()"></div>
      <div class="time-spend">{{timeSpend}}</div>
    </div>
    <div class="game">
      <div v-for="(s,i) in mapData" :key="i">
        <div
          v-for="(b,index) in s"
          :key="index"
          class="block"
          :class="mapRevealed[i][index] === 'F'? 'shadow': 'block-'+mapRevealed[i][index]"
          @click="openBlock(i,index)"
        ></div>
      </div>
    </div>

    <div class="score-input" v-if="win">
      <div class="input-box">
        <div>请问尊姓大名：</div>
        <input v-model="playerName" />
        <div>成绩：{{timeSpend}}</div>
        <button @click="postScore()">确定</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
const MAP_SIZE = 16;
const MINE_COUNT = 40;

export default {
  data() {
    return {
      mapData: [],
      mapRevealed: [],
      timeSpend: 0,
      timer: 0,
      gameOver: false,
      win: false,
      playerName: ""
    };
  },
  mounted() {
    window.console.log("this is a new game");
    this.restart();
  },
  methods: {
    drawMap(x, y) {
      let mine_count_left = MINE_COUNT;
      let blank_count_left = MAP_SIZE * MAP_SIZE - MINE_COUNT - 1;

      for (let ix = 0; ix < MAP_SIZE; ix++) {
        for (let iy = 0; iy < MAP_SIZE; iy++) {
          if (x == ix && y == iy) {
            this.mapData[ix][iy] = "S";
          } else if (
            (Math.random() < 0.16 && mine_count_left > 0) ||
            blank_count_left == 0
          ) {
            this.mapData[ix][iy] = "M";
            mine_count_left--;
          } else {
            this.mapData[ix][iy] = "S";
            blank_count_left--;
          }
        }
      }

      for (let ix = 0; ix < MAP_SIZE; ix++) {
        for (let iy = 0; iy < MAP_SIZE; iy++) {
          this.fillNumber(ix, iy, this.mapData[ix][iy]);
        }
      }
    },
    restart() {
      // 覆盖地图
      let mapRevealed = [];
      for (let i = 0; i < MAP_SIZE; i++) {
        mapRevealed.push(new Array(MAP_SIZE).fill("F"));
      }

      let mapData = [];
      for (let ix = 0; ix < MAP_SIZE; ix++) {
        let tempArr = [];
        for (let iy = 0; iy < MAP_SIZE; iy++) {
          tempArr.push("");
        }
        mapData.push(tempArr);
      }

      this.mapData = mapData;
      this.mapRevealed = mapRevealed;

      this.gameOver = false;
      this.win = false;
      this.timeSpend = 0;

      if (this.timer) {
        clearInterval(this.timer);
        this.timer = 0;
      }
      // this.timer = 0;
    },
    getNearby(x, y) {
      let tmp = [
        [x - 1, y - 1],
        [x - 1, y],
        [x - 1, y + 1],
        [x, y - 1],
        [x, y + 1],
        [x + 1, y - 1],
        [x + 1, y],
        [x + 1, y + 1]
      ];
      let ans = [];

      tmp.forEach(pos => {
        if (
          pos[0] >= 0 &&
          pos[0] < MAP_SIZE &&
          pos[1] >= 0 &&
          pos[1] < MAP_SIZE
        ) {
          ans.push(pos);
        }
      });

      return ans;
    },
    fillNumber(x, y, val) {
      if (val == "M") {
        return;
      }
      let nearby = this.getNearby(x, y);
      let number = 0;
      nearby.forEach(pos => {
        if (this.mapData[pos[0]][pos[1]] === "M") {
          number++;
        }
      });

      this.mapData[x][y] = number;
    },
    openBlock(x, y) {
      // 已揭示则不继续循环
      if (this.mapRevealed[x][y] !== "F") {
        return;
      }
      // gameOver则不响应
      if (this.gameOver) {
        return;
      }
      // 开始计时
      if (!this.timer) {
        this.timer = setInterval(() => {
          this.timeSpend++;
        }, 1000);

        this.drawMap(x, y);
      }

      // 监听数组变化的方法
      this.$set(this.mapRevealed[x], y, this.mapData[x][y]);

      let val = this.mapData[x][y];
      if (val === "M") {
        // game over
        this.gameOver = true;
        this.revealMine();
        this.$set(this.mapRevealed[x], y, "R");
        clearInterval(this.timer);
        this.timer = 0;
        return;
      } else if (val === 0) {
        let nearby = this.getNearby(x, y);
        nearby.forEach(val => {
          this.openBlock(val[0], val[1]);
        });
      } else {
        // do nothing, just show the number
      }

      // 判断游戏是否结束
      let all = this.mapRevealed.reduce((pre, val) => {
        return pre.concat(val);
      });
      let leftCount = all.filter(val => {
        return val === "F";
      });

      if (leftCount.length <= MINE_COUNT) {
        this.win = true;
        // 停止倒计时 结算成绩
        clearInterval(this.timer);
        this.timer = 0;
      }
    },
    revealMine() {
      this.mapData.forEach((arr, x) => {
        arr.forEach((val, y) => {
          if (val === "M") {
            this.mapRevealed[x][y] = "M";
          }
        });
      });
    },
    postScore() {
      axios
        .post("http://mooncake.migame.xyz:8004/score", {
          name: this.playerName,
          score: this.timeSpend
        })
        .then(() => {
          this.restart();
        });
    }
  }
};
</script>

<style lang="less">
.task {
  margin: auto auto 10px;
}
.smile {
  display: inline-block;
  width: 64px;
  height: 64px;
  background: url("../assets/smile.png");
  background-size: 100% 100%;
  background-repeat: no-repeat;
  margin-right: 10px;
  vertical-align: middle;
}
.time-spend {
  display: inline-block;
  width: 128px;
  height: 64px;
  background-color: #888;
  color: red;
  font-size: 24px;
  line-height: 64px;
  vertical-align: middle;
}
.game-over {
  background-image: url("../assets/over.png");
}
.win {
  background-image: url("../assets/win.png");
}
.game {
  margin: auto;
  width: 640px;
  height: 640px;
  background: #999;
  .block {
    display: block;
    float: left;
    box-sizing: border-box;
    border: 2px solid #888;
    width: 40px;
    height: 40px;
    background: #999;
    line-height: 40px;
  }
  .shadow {
    background: url("../assets/shadow.png");
    background-size: 100% 100%;
  }

  .generate-block(8);

  .generate-block(@n, @i: 0) when (@i =< @n) {
    .block-@{i} {
      background: url("../assets/@{i}.png");
      background-repeat: no-repeat;
      background-size: 100% 100%;
    }
    .generate-block(@n, (@i + 1));

    .block-M {
      background: url("../assets/mine.png");
      background-repeat: no-repeat;
      background-size: 100% 100%;
    }

    .block-R {
      background: url("../assets/red.png");
      background-repeat: no-repeat;
      background-size: 100% 100%;
    }
  }
}

.score-input {
  position: fixed;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(1, 1, 1, 0.8);
  .input-box {
    position: absolute;
    width: 500px;
    height: 200px;
    background: #eee;
    left: 50%;
    top: 50%;
    margin-left: -250px;
    margin-top: -100px;
  }
}
</style>