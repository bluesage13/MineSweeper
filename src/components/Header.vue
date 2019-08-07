<template lang="html">
  <div class="header">
    <button type="button" class="btn btn-primary headerSection" @click="startGame">New Game</button>
    <button type="button" class="btn btn-danger headerSection" @click="endGame">End Game</button>
    <div class="headerSection flag" v-if="!gameStatus && game != 0">
      GAME OVER!!
    </div>
    <div class="headerSection flag" v-if="gameStatus">
      <div class="flagText">Flags</div>
      <div class="flagCount">{{flagsLeft}}</div>
    </div>
  </div>
</template>

<script>
import {dataBus} from '../main';

export default {
  props:['game', 'gameStatus'],
  data: function(){
    return {
      flagsLeft : 10
    };
  },
  methods: {
    startGame(){
      this.$emit('startGame');
    },
    endGame(){
      this.$emit('endGame');
    }
  },
  created(){
    dataBus.$on('flagPlaced', (data) =>
    {
      this.flagsLeft = data;
    }
  );
  }
}
</script>

<style lang="css" scoped>
.header {
  height:100px;
  background-color: #ecf0f1;
  display:flex;
  align-content: center;
  justify-content:flex-start;
  padding-top: 25px;
}
.headerSection {
  height: 50px;
  margin-left: 10%;
}
.flag {
  font-family: "Helvetica Neue", "Arial", sans-serif;
  font-weight: bold;
  color: #2c3e50;
  margin-left: auto;
  margin-right: 30px;
  display: inline-block;
  width: 50px;
  text-align: center;
}
</style>
