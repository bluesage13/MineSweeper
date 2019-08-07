<template lang="html">
  <div class="minefield-container" :id="game">
    <div class="gameOver" v-if="!gameStatus && game != 0">
      <h1>GAME OVER!!!!</h1>
    </div>
    <div class="gameOver" v-if="hasWon && game != 0 ">
      <h1>YOU WON!!!</h1>
    </div>
    <div
      v-for="i in 100"
      :id="i-1"
      class="tile"
      :class="{
              flag: hasFlag(i-1),
              exposed: exposedStatus[i-1],
              even: tileColor(i-1) == true,
              odd : tileColor(i-1) == false /*&& exposedStatus[i-1] != true*/
              }"
      @click="handleClick"
      @click.right.prevent="placeFlag(i-1)"
      >
      <p v-if="exposedStatus[i-1] && !hasZero(i-1)" >{{getTileData(i-1)}}</p>
      <!-- <p v-if="i-1 == i-1" >{{getTileData(i-1)}}</p> -->
    </div>
  </div>
</template>

<script>
import {dataBus} from '../main';

var isExposedArr = [];
for(var j = 0; j < 100; j++){
  isExposedArr.push(false);
}

export default {
  props:['gameStatus','game','mines','tiles', 'hasWon'],
  data: function(){
    return {
      exposedStatus: [],
      visited: [],
      flagCount: 10,
      totalExposed: 0,
      score : 0,
      flagId: [],
      cnt: 0
    }
  },

  methods: {
    exposeDFS(r, c, id){
      if(this.visited[r][c] || this.tiles[r][c] == -1){
        return;
      }
      this.visited[r][c] = true;
      if(this.tiles[r][c] > 0){
        this.exposedStatus.splice(id,1,true);
        return;
      }
      this.exposedStatus.splice(id,1,true);
      var transition = [[-1,0],[1,0],[0,-1],[0,1]];
      for(var x = 0; x < transition.length; x++){
        var n_r = r + transition[x][0];
        var n_c = c + transition[x][1];
        var n_id = n_r * this.tiles.length + n_c;
        if(n_r >= 0 && n_c >= 0 && n_r < this.tiles.length && n_c < this.tiles[0].length){
          this.exposeDFS(n_r, n_c,n_id);
        }
      }
      return;

    },
    initVisited(){
      this.visited = [];
      for(var i = 0; i < this.tiles.length; i++){
        var row = []
        for(var j = 0; j < this.tiles[0].length; j++){
          row.push(false);
        }
        this.visited.push(row);
      }
    },
    calculateVisited(){
      this.totalExposed = 0;
      for(var i = 0; i < this.visited.length; i++){
        for(var j = 0; j < this.visited[0].length; j++){
          if(this.visited[i][j]){
            this.totalExposed += 1;
          }
        }
      }
    },
    handleClick(event){
      var id = event.target.id;
      if(this.gameStatus && this.exposedStatus[id] != true){

        //click on safe mine
        var tile_r = Math.floor(id/10);
        var tile_c = id % 10;
        var idHasMine = false;
        this.mines.forEach((el) => {
          if(el == id){
            idHasMine = true;
          }
        });
        //Clicked on mine
        if(idHasMine){
          console.log("Game Over");
          this.gameStatus = false;
          this.$emit('gameEnded');
        }else{
          this.initVisited();
          this.exposeDFS(tile_r,tile_c,id);
        }
        this.calculateVisited();
      }
      if(this.score == 10){
        this.checkIfWon();
      }
    },
    placeFlag(id){
      if(this.exposedStatus[id] != true && this.hasFlag(id) != true){
        if(this.flagCount != 0){
          this.flagCount -= 1;
          dataBus.$emit('flagPlaced', this.flagCount);
        }
        var idHasMine = false;
        this.mines.forEach((el) => {
          if(el == id){
            idHasMine = true;
          }
        });
        if(idHasMine){
          this.score += 1;
        }
        this.flagId.push(id);
      }
      else if(this.hasFlag(id) == true){
        if(this.mines.includes(id)){
          this.score -=1;
        }
        // this.flagId = this.flagId.splice(id,1);
        //this.flagId.splice(id,1);
        this.flagId = this.removeFromFlagId(id);
        this.flagCount += 1;
        dataBus.$emit('flagPlaced', this.flagCount);
      }
      //console.log("Score-<",this.score);
      if(this.score == 10){
        this.checkIfWon();
      }
    },
    checkIfWon(){
      var count = 0;
      for(var j = 0; j < this.exposedStatus.length; j++){
        if(this.exposedStatus[j]){
          count += 1;
        }
      }
      if(count >= 90){
        //dataBus.$emit('gameWon');
        this.$emit('gameWon');
        console.log("Won!");
      }
    },
    removeFromFlagId(id){
      var temp = [];
      for(var i = 0; i < this.flagId.length; i++){
        if(this.flagId[i] != id)
          temp.push(this.flagId[i]);
      }
      return temp;
    }
    ,
    hasFlag(id){
      if(this.flagId.length == 0){
        return false;
      }
      return this.flagId.includes(id);
    },
    hasZero(id){
      var tile_r = Math.floor(id/10);
      var tile_c = id % 10;
      return this.tiles[tile_r][tile_c] == 0;
    },
    tileColor(id){
      var rem = ((id%10) + (Math.floor(id/10))%2)%2;
      if(rem == 0){
        return true;
      }else{
        return false;
      }
    },
    getTileData(id){
      if(this.tiles.length != 0){
        var tile_r = Math.floor(id/10);
        var tile_c = id % 10;
        return this.tiles[tile_r][tile_c];
      }

    }
  },
  created(){
    for(var j = 0; j < 100; j++){
      this.exposedStatus.push(false);
    }
  }
}
</script>

<style lang="css" scoped>
.minefield-container{
  height: 500px;
  width: 500px;
  background-color: #34495e;
  margin:auto;
}
.tile{
  height:50px;
  width: 50px;
  background-color: #3498db;
  float:left;
  display:flex;
  text-align: center;
  justify-content: center;
  font-size: 30px;
  font-family: "Helvetica Neue", "Arial", sans-serif;
  font-weight: bold;
  color: #2c3e50;
}
.tile:hover{
  background-color: #adffcf;
}
.even{
  background-color: #2ecc71;
}
.odd{
  background-color: #1abc9c;
}
.exposed{
  background-color: #ecf0f1;
}
.flag{
  background-color: #d35400;
}
.gameOver{
  text-align: center;
  justify-content: center;
  display: flex;
  padding: 200px;
  font-family: "Helvetica Neue", "Arial", sans-serif;
  font-weight: bold;
  color: #ecf0f1;
  position: fixed;
  width: 500px;
  height: 500px;
  background-color: rgba(0,0,0,0.5);
  z-index: 2;
  cursor: pointer;
}
</style>
