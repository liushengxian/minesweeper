<template>
  <div id="app">
    <h1>MineSweeper Misan</h1>
    <div class="high-score">
      <div class="music-block" :class="musicPlaying?'play':''" @click="toggleMusic()"></div>
      <ul>
        <li v-for="(v,i) in rankList" :key="i" class="piece" v-show="i<3">
          <div>排名：{{i+1}}</div>
          <div>名称: {{v.name}}</div>
          <div>所用时间: {{v.score}}</div>
        </li>
      </ul>
    </div>
    <Game></Game>
    <audio
      ref="music"
      src="https://misanya-1252867445.cos.ap-shanghai.myqcloud.com/%CE%BC's%20-%20%E3%82%BF%E3%82%AB%E3%83%A9%E3%83%A2%E3%83%8E%E3%82%BA.mp3"
      loop
      autoplay
    ></audio>
  </div>
</template>

<script>
import axios from "axios";
import Game from "./components/Game.vue";

export default {
  name: "app",
  components: {
    Game
  },
  data() {
    return {
      rankList: [],
      musicPlaying: true
    };
  },
  mounted() {
    axios.get("http://mooncake.migame.xyz:8004/score").then(val => {
      window.console.log(val.data);
      this.rankList = val.data;
      // this.rankList.push({
      //   name:'2',
      //   score: 3
      // })
      this.rankList.sort((a, b) => {
        return a.score - b.score;
      });
    });

    setTimeout(() => {
      this.$refs.music.play();
    }, 1000);
  },
  methods: {
    toggleMusic() {
      if (this.$refs.music.paused) {
        this.$refs.music.play();
        this.musicPlaying = true;
      } else {
        this.$refs.music.pause();
        this.musicPlaying = false;
      }
    }
  }
};
</script>

<style lang="less">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
body {
  background: #555;
}

.high-score {
  position: fixed;
  left: 0;
  top: 300px;
  width: 500px;
  text-align: left;
  ul {
    li {
      list-style: none;
    }
  }
  .piece div {
    display: inline-block;
    margin-right: 15px;
  }
}

@keyframes circle {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.music-block {
  background: url("./assets/music.png");
  background-size: 100% 100%;
  width: 40px;
  height: 40px;
  margin: auto;
}

.play {
  animation: circle linear 3s infinite;
}
</style>
