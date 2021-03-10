<template>
  <div id="app">
    <div class="header margin-5 padding-10 center solid-border">
      <span><h1>Welcome to the Snake Vue Game</h1></span>
    </div>
    <div class="content margin-5 padding-10">
      <div class="frame margin-5 padding-5">
        <div class="game margin-right-10">
          <h1>Frame</h1>
          <Row
            v-for="row in getblocks"
            v-bind:key="row.rownum"
            :rownum="row.rownum"
            :colcount="row.cells.length"
          ></Row>
          <div class="overlay-text padding-10" v-if="gameend">
            <span>GAME OVER !!!</span>
          </div>
        </div>
      </div>
      <div class="controls margin-5 padding-5">
        <h1>Controls</h1>
        <br /><br />
        <div class="score">
          <span><b>Score:</b> {{ score }}</span>
        </div>
        <br /><br />
        <div class="grid padding-10">
          <div class="button" v-on:click="setdirection(1)">Left</div>
        </div>
        <div class="grid padding-10">
          <div class="button margin-bottom-10" v-on:click="setdirection(2)">
            Up
          </div>
          <div class="button" v-on:click="setdirection(4)">Down</div>
        </div>
        <div class="grid padding-10">
          <div class="button" v-on:click="setdirection(3)">Right</div>
        </div>
        <div class="row">
          <div class="button" v-on:click="newgame()">New Game</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Row from "@/components/Row.vue";

var rowscount = 20,
  colscount = 20,
  startlength = 5,
  defaultinterval = 250,
  mininterval = 50,
  intervaldecrementval = 10; //Value to decrement interval by

export default {
  name: "Snake",
  components: {
    Row,
  },
  data: () => {
    return {
      rowscount: rowscount,
      colscount: colscount,
      blocks: null,
      snake: null,
      fruit: null,
      dir: 1, //Dir 1-left, 2-top, 3-right, 4-bottom
      gameend: false,
      timer: false, //timeout variable containing timeout object
      interval: null, //interval in ms between step refresh
      score: null,
    };
  },
  computed: {
    getblocks: function () {
      return this.blocks;
    },
    getsnake: function () {
      return this.snake;
    },
  },
  methods: {
    updateblocks: function () {
      var cur = this.snake;
      //Set all blocks unshaded
      this.clearblocks();
      while (cur != null) {
        this.blocks[cur.x].cells[cur.y].shaded = true;
        cur = cur.next;
      }
      this.blocks[this.snake.x].cells[this.snake.y].head = true;
      this.blocks[this.fruit.x].cells[this.fruit.y].fruit = true;
      this.blocks[this.fruit.x].cells[this.fruit.y].shaded = true;
    },
    clearblocks: function () {
      for (var i = 0; i < rowscount; i++) {
        for (var j = 0; j < colscount; j++) {
          this.blocks[i].cells[j].shaded = false;
          this.blocks[i].cells[j].head = false;
          this.blocks[i].cells[j].fruit = false;
        }
      }
    },
    newgame: function () {
      console.log("Creating new game");
      //Create the snake
      this.snake = {
        x: Math.floor(Math.random() * this.rowscount), //Row position
        y: Math.floor(Math.random() * this.colscount), // Col position
        head: true,
      };
      this.setdirection(Math.floor(Math.random() * 5));
      for (var i = 0; i < startlength - 1; i++) {
        this.addblock();
      }
      this.createfruit();
      this.updateblocks();
      this.interval = defaultinterval;
      this.gameend = false;
      this.score = 0;
      this.refreshtimer();
    },
    refreshtimer: function () {
      clearInterval(this.timer);
      this.timer = setInterval(this.movestep, this.interval);
    },
    decrementinterval: function () {
      this.interval -= intervaldecrementval;
      if (this.interval < mininterval) {
        this.interval = mininterval;
      }
    },
    createfruit: function () {
      this.fruit = {
        x: Math.floor(Math.random() * this.rowscount), //Row position
        y: Math.floor(Math.random() * this.colscount), // Col position
      };
      while (this.fruit.x == this.snake.x && this.fruit.y == this.snake.y) {
        this.fruit.x = Math.floor(Math.random() * this.rowscount); //Row position
        this.fruit.y = Math.floor(Math.random() * this.colscount); // Col position
      }
    },
    addblock: function () {
      var next = this.getnextposition(this.snake);
      next.next = this.snake;
      next.head = true;
      this.snake.head = false;
      this.snake = next;
    },
    movestep: function () {
      if (!this.gameend) {
        var next = this.getnextposition(this.snake);
        var cur = this.snake;
        while (cur.next != null && cur.next.next != null) {
          cur = cur.next;
        }
        cur.next = null;
        next.next = this.snake;
        this.snake = next;
        this.updateblocks();
        this.gameend = this.checkcollision();
        if (this.checkreachfruit()) {
          this.createfruit();
          this.addblock();
          this.score++;
          this.decrementinterval();
          this.refreshtimer();
        }
      }
    },
    setdirection: function (dir) {
      //Dir 1-left, 2-top, 3-right, 4-bottom
      this.dir = dir;
    },
    checkcollision: function () {
      var cur = this.snake.next;
      while (cur) {
        if (cur.x == this.snake.x && cur.y == this.snake.y) {
          return true;
        }
        cur = cur.next;
      }
      return false;
    },
    checkreachfruit: function () {
      return this.snake.x == this.fruit.x && this.snake.y == this.fruit.y;
    },
    getnextposition: function (cur) {
      var nextcell = {
        //Set x and y positions for the next cell based on direction specified. Next will point to the cur value
        x: cur.x,
        y: cur.y,
      };
      switch (this.dir) {
        case 0: //Consider 0 as 1
        case 1: //left
          nextcell.y--;
          break;
        case 2:
          nextcell.x--;
          break;
        case 3:
          nextcell.y++;
          break;
        case 4:
          nextcell.x++;
          break;
      }
      if (nextcell.x < 0) {
        nextcell.x = this.rowscount - 1;
      } else if (nextcell.x >= this.rowscount) {
        nextcell.x = 0;
      }
      if (nextcell.y < 0) {
        nextcell.y = this.colscount - 1;
      } else if (nextcell.y >= colscount) {
        nextcell.y = 0;
      }
      return nextcell;
    },
  },
  mounted() {
    //define the block structure.
    //Rows have 2 properties: rownum, cells
    //Cells have 2 properties: colnum, shaded, fruit
    this.blocks = [];
    for (var i = 0; i < rowscount; i++) {
      var row = {
        rownum: i,
        cells: [],
      };
      for (var j = 0; j < colscount; j++) {
        var cell = {
          colnum: j,
          shaded: false,
          fruit: false,
          head: false,
        };
        row.cells.push(cell);
      }
      this.blocks.push(row);
    }

    window.addEventListener("keydown", (e) => {
      switch (e.key) {
        case "N":
          if (e.shiftKey) {
            this.newgame();
          }
          break;
        case "ArrowLeft":
          e.preventDefault();
          this.setdirection(1);
          break;
        case "ArrowUp":
          e.preventDefault();
          this.setdirection(2);
          break;
        case "ArrowRight":
          e.preventDefault();
          this.setdirection(3);
          break;
        case "ArrowDown":
          e.preventDefault();
          this.setdirection(4);
          break;
      }
    });

    //Create new game on load
    this.newgame();
  },
};
</script>