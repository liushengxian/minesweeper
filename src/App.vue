<template>
  <div id="app">
    <h1>MineSweeper Misan</h1>
    <ul>
      <li v-for="i in rankList" :key="i">rank</li>
    </ul>
    <div>Play New Game</div>
    <Game></Game>
  </div>
</template>

<script>
import Game from './components/Game.vue'

export default {
  name: 'app',
  components: {
    Game
  },
  data(){
    return {
      rankList:[1,2,3]
    }
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
</style>
