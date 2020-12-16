<template>
  <div id="app">
    <audio id="a" src=""></audio>
    <div class="game-board">
      <div class="box-1" @click="registerClick($event)" data-id="1"></div>
      <div class="box-2" @click="registerClick($event)" data-id="2"></div>
      <div class="box-3" @click="registerClick($event)" data-id="3"></div>
      <div class="box-4" @click="registerClick($event)" data-id="4"></div>
    </div>
    <div>
      <div>
        <p v-if="gameIsOn">Раунд:{{ round }}</p>
        <p v-if="userLost && !gameIsOn">Вы проиграли!</p>
      </div>
      <div>
        <h3>Выберите сложность</h3>
      </div>
      <div>
        <label for="easy">Легкий</label>
        <input type="radio" value="1500" checked v-model="mode" />
      </div>
      <div>
        <label for="easy">Средний</label>
        <input type="radio" value="600" v-model="mode" />
      </div>
      <div>
        <label for="easy">Тяжелый</label>
        <input type="radio" value="400" v-model="mode" />
      </div>
    </div>
    <button @click="startGame" v-if="!userLost && !gameIsOn">Начать</button>
    <button @click="startGame" v-if="userLost && !gameIsOn">
      Начать сначала
    </button>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      mode: 1500,
      gameSequence: [],
      copy: [],
      clicked: null,
      round: 0,
      userTurn: false,
      userSquence: [],
      gameIsOn: false,
      userLost: false,
    };
  },
  computed: {
    sequenceToGuess: function () {
      return [...this.gameSequence];
    },
  },
  methods: {
    playAudio(item) {
      let a = new Audio(require(`./assets/sounds/${item.dataset.id}.mp3`));
      console.log('123')
      a.play();
    },
    flash(item) {
      return new Promise((resolve) => {
        item.classList.add("lit");
        this.playAudio(item);
        setTimeout(() => {
          item.classList.remove("lit");
          setTimeout(() => resolve(), 250);
        }, parseInt(this.mode));
      });
    },
    playSequence: async function () {
      this.userTurn = false;
      let s = this.gameSequence.map((el) => {
        return document.querySelector(`[data-id="${el}"]`);
      });
      for (const box of s) {
        await this.flash(box);
      }
      this.userTurn = true;
    },
    randomNumber() {
      return Math.floor(Math.random() * 4 + 1);
    },
    startGame() {
      this.gameSequence = [];
      this.gameIsOn = true;
      this.newRound();
    },
    gameOver() {
      this.round = 0;
      this.gameIsOn = false;
      this.userTurn = false;
      this.userLost = true;
    },
    newRound() {
        //setup listeners
      let elems = document.querySelectorAll(`[data-id]`);
      elems.forEach((elem) => {
      elem.addEventListener("click", this.userClick);
      elem.addEventListener("mousedown", this.userMouseDown);
      elem.addEventListener("mouseup", this.userMouseUp);
      });
      this.round++;
      this.userSquence = [];
      this.gameSequence.push(this.randomNumber());
      this.playSequence();
    },
    userClick(e) {
      console.log(e)
      this.playAudio(e.target);
    },
    userMouseDown(e) {
      e.target.classList.add("lit");
    },
    userMouseUp(e) {
      e.target.classList.remove("lit");
    },
    registerClick(e) {
      if (this.userTurn === true) {
        this.userSquence.push(parseInt(e.target.dataset.id));
        this.clicked = e.target.dataset.id;
        this.checkLose();
      }
    },
    checkLose() {
      let expectedNumber = this.sequenceToGuess.shift();
      if (expectedNumber == this.clicked) {
        if (this.sequenceToGuess.length === 0) {
          let elems = document.querySelectorAll(`[data-id]`);
          elems.forEach((elem) => {
            elem.removeEventListener("click", this.userClick);
            elem.removeEventListener("mousedown", this.userMouseDown);
            elem.removeEventListener("mouseup", this.userMouseUp);
          });
          this.newRound();
        }
      } else {
        alert("game over!");
        this.gameOver();
      }
    },
  },
};
</script>

<style lang="scss">
$border_color: 1px solid black;
* {
  box-sizing: border-box;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.game-board {
  margin: 1rem auto;
  display: flex;
  width: 200px;
  height: 200px;
  flex-wrap: wrap;
  .box-1 {
    background-color: rgba(0, 0, 255, 1);
    opacity: 0.5;
    &:hover {
      border: $border_color;
    }
  }
  .box-2 {
    background-color: rgba(255, 0, 0, 1);
    opacity: 0.5;
    &:hover {
      border: $border_color;
    }
  }
  .box-3 {
    background-color: rgba(255, 238, 0, 1);
    opacity: 0.5;
    &:hover {
      border: $border_color;
    }
  }
  .box-4 {
    background-color: rgba(9, 255, 0, 1);
    opacity: 0.5;
    &:hover {
      border: $border_color;
    }
  }
  .lit {
    opacity: 1;
  }
}
div [data-id] {
  width: 100px;
  height: 100px;
}
</style>
