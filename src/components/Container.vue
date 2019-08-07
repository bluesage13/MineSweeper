<template lang="html">
  <div class="gameContainer" :key="gameNumber">
    <my-header
      @startGame='init'
      @endGame='end'
      :game="gameNumber"
      :gameStatus="isGameOn"
      ></my-header>
    <mine-field-container
    @tileClicked='handleTileClick'
    @gameEnded='end'
    @gameWon='won'
    :gameStatus="isGameOn"
    :game="gameNumber"
    :mines="mines"
    :tiles="tiles"
    :hasWon="hasWon"></mine-field-container>
  </div>
</template>

<script>
import Header from './Header.vue';
import MineField from './MineField.vue';

export default {
  data: function(){
    return {
      mines : [],
      tiles : [],
      length: 10,
      isGameOn: false,
      gameNumber: 0,
      hasWon: false
    }
  }
  ,
  methods: {
    initTiles(){
      this.tiles = [];
      for(var k = 0; k < this.length; k++){
        var row = [];
        for(var l = 0; l < this.length; l++){
          row.push(0);
        }
        this.tiles.push(row);
      }
    },
    init(){
      this.gameNumber += 1;
      this.isGameOn = true;
      this.hasWon= false;
      this.initTiles();
      this.mines = [];
      // for(var i = 0; i < this.length || this.mines.length < this.length; i++){
      for(var i = 0; this.mines.length < this.length; i++){
        var mineInd = Math.floor(Math.random() * this.length * this.length)
        if(this.mines.includes(mineInd)){
          continue;
        }
        this.mines.push(mineInd);
        var r = Math.floor(mineInd / 10);
        var c = mineInd % 10;
        this.tiles[r][c] = -1;
        var transition=[[-1,-1],[-1,0],[-1,1],[0,-1],[0,1],[1,-1],[1,0],[1,1]];
        for(var x = 0; x < transition.length; x++){
          var n_r = r + transition[x][0];
          var n_c = c + transition[x][1];
          if(n_r >= 0 && n_c >= 0 && n_r < this.tiles.length && n_c < this.tiles[0].length && this.tiles[n_r][n_c] != -1){
            this.tiles[n_r][n_c] += 1;
          }
        }
      }
      console.log(this.mines.length);
      console.log(this.mines);
    },
    end(){
      this.isGameOn = false;
    },
    won(){
      this.hasWon = true;
    },
    handleTileClick(id){
      if(this.isGameOn){
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
          this.isGameOn = false;
        }
      }
    }
  },
  components: {
    myHeader : Header,
    mineFieldContainer : MineField
  }
}
</script>

<style lang="css" scoped>
  .gameContainer{
    height: 600px;
    width: 600px;
    margin: 5% auto;
    background-color: #2c3e50;
  }
</style>
