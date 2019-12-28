<template>

  <div id="app">
    <table>
      <tr v-bind:key="y" v-for="y in range(startY, endY)">
        <td :class="{'snake-body' : isBodyCell(x, y), 'snake-head' : isHeadCell(x, y), 'eat-block' : isEatCell(x, y)}" v-bind:key="x" v-for="x in range(startX, endX)"></td>
      </tr>
    </table>
    <button @click="startNewGame">Start new game</button>
    <h1>Очки: {{score}}</h1>
  </div>
</template>

<script>
export default {
  name: 'app',
  mounted: function () {
    var that = this;
    this.startNewGame();
    window.addEventListener('keydown', function(e) {
      that.handleGlobalKeyDown(e)
    });
  },
  data(){
    return {
      inputedDirection  : null,
      direction         : null,
      snakeBlocks       : [],
      headBlock         : null,
      eatBlock          : null,
      ticking           : null,
      holdTail          : null,
      startX            : null,
      startY            : null,
      endX              : null,
      endY              : null,
    };
  },
  computed: {
    score(){
        return this.snakeBlocks.length - 1;
    },
  },
  methods: {
    startNewGame(){
      clearInterval(this.ticking);
      let proto = {
        inputedDirection  : 'right',
        direction         : 'right',
        snakeBlocks       : [{x: 5, y: 6}],
        headBlock         : {x: 6, y: 6},
        eatBlock          : {x: 1, y: 4},
        ticking           : null,
        holdTail          : false,
        startX            : 0,
        startY            : 0,
        endX              : 32,
        endY              : 10,
      };
      for (var prop in this.$data) {
        if (Object.prototype.hasOwnProperty.call(this.$data, prop)) {
          this[prop] = proto[prop];
        }
      }
      this.setNewEatBlock();
      this.ticking = setInterval(this.tick, 80);
    },
    range(start, end) {
      var foo = [];
      for (var i = start; i <= end; i++) {
        foo.push(i);
      }
      return foo;
    },
    rand(min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min);
    },
    moveTail(){
      let firstBlock = this.headBlock;
      for(let i = 0; i < this.snakeBlocks.length; i++){
        let firstPosition = {... this.snakeBlocks[i]};
        this.snakeBlocks[i].x = firstBlock.x;
        this.snakeBlocks[i].y = firstBlock.y;
        firstBlock = firstPosition;
      }
    },
    moveRight(){
      this.headBlock.x++;
      if(this.headBlock.x > this.endX) {
        this.headBlock.x = this.startX;
      }
    },
    moveLeft(){
      this.headBlock.x--;
      if(this.headBlock.x < this.startX) {
        this.headBlock.x = this.endX;
      }
    },
    moveTop(){
      this.headBlock.y--;
      if(this.headBlock.y < this.startY) {
        this.headBlock.y = this.endY;
      }
    },
    moveDown(){
      this.headBlock.y++;
      if(this.headBlock.y > this.endY) {
        this.headBlock.y = this.startY;
      }
    },
    tick(){
      this.direction = this.inputedDirection;
      if(!this.holdTail)
        this.moveTail();
      this.holdTail = false;
      switch (this.direction) {
        case 'right'  : this.moveRight();break;
        case 'left'   : this.moveLeft();break;
        case 'top'    : this.moveTop();break;
        case 'down'   : this.moveDown();break;
      }
      this.checkCollision();
      let eaten = this.tryEatBlock();
      if(eaten){
        this.holdTail = true;
        this.addNewTailBlock({... this.headBlock});
      }
    },
    addNewTailBlock(proto){
      this.snakeBlocks.unshift({x: proto.x, y: proto.y});
    },
    checkCollision(){
      if(this.isBodyCell(this.headBlock.x, this.headBlock.y)){
        clearInterval(this.ticking);
      }
    },
    setNewEatBlock(){
      var that = this;
      let matrix = [];
      for(let x = this.startX; x <= this.endX; x++){
        for(let y = this.startY; y <= this.endY; y++){
          matrix[matrix.length] = {x: x, y: y};
        }
      }
      let emptyBlocks = matrix.filter(function(block){
        let x = block.x;
        let y = block.y;
        return !(Boolean(that.snakeBlocks.find(function(block2){
          return block2.x === x && block2.y === y;
        })) || (x === that.headBlock.x && y === that.headBlock.y));
      });
      this.eatBlock = emptyBlocks[this.rand(0, emptyBlocks.length)];
      console.log(this.eatBlock);
    },
    tryEatBlock(){
      if(this.headBlock.x === this.eatBlock.x && this.headBlock.y === this.eatBlock.y){
        this.setNewEatBlock();
        return true;
      }
      return false;
    },
    handleGlobalKeyDown(event){
      switch (event.key.toUpperCase()) {
        case 'A' : if(this.direction === 'right') break;  this.inputedDirection = 'left';  break;
        case 'W' : if(this.direction === 'down')  break;  this.inputedDirection = 'top';   break;
        case 'S' : if(this.direction === 'top')   break;  this.inputedDirection = 'down';  break;
        case 'D' : if(this.direction === 'left')  break;  this.inputedDirection = 'right'; break;
        default  : break;
      }
    },
    isBodyCell(x, y){
      return this.snakeBlocks && Boolean(this.snakeBlocks.find(function(block){
        return block.x === x && block.y === y;
      }));
    },
    isHeadCell(x, y){
        return this.headBlock && x === this.headBlock.x && y === this.headBlock.y;
    },
    isEatCell(x, y){
        return this.eatBlock && x === this.eatBlock.x && y === this.eatBlock.y;
    },
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  font-size: 10px;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
table {
  border: 1px solid #c1c1c1;
  border-collapse: collapse;
}
td {
  width: 18px;
  height: 18px;
  outline: 1px solid #f5f5f5;
}
td.snake-body {
  background-color: #3a7eff;
  border-radius: 20px;
}
td.snake-head {
  background-color: #0f13ff;
  border-radius: 20px;
}
td.eat-block {
  background-color: #91ff92;
  border-radius: 20px;
}
</style>
