<template>
  <div id="app">
    <h1>MineSweeper Misan</h1>
    <ul>
      <li v-for="(v,i) in rankList" :key="i" class="piece" v-show="i<3">
        <div>排名：{{i+1}}</div>
        <div>名称: {{v.name}}</div>
        <div>所用时间: {{v.score}}</div>
      </li>
    </ul>
    <Game></Game>
  </div>
</template>

<script>
import axios from "axios"
import Game from './components/Game.vue'

export default {
  name: 'app',
  components: {
    Game
  },
  data(){
    return {
      rankList:[]
    }
  },
  mounted(){
    axios.get('http://mooncake.migame.xyz:8004/score').then(val =>{
      window.console.log(val.data);
      this.rankList = val.data;
      // this.rankList.push({
      //   name:'2',
      //   score: 3
      // })
      this.rankList.sort((a,b)=>{
        return a.score - b.score;
      })

    })
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
body{
  background: #555;
}

.piece div{
  display: inline-block;
  margin-right: 15px;
}
</style>
