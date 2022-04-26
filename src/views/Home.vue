<template>
  <div class="main">
    <div class="main-title">贪吃蛇</div>
    <div class="main-body">
      <div class="game" ref="game">
        <div class="snake">
          <div
            v-for="(item, index) in snake"
            :key="index"
            :style="{ left: item[0] + 'px', top: item[1] + 'px' }"
          ></div>
        </div>
        <div
          class="food"
          :style="{ left: foodX + 'px', top: foodY + 'px' }"
        ></div>
      </div>
      <div class="score">
        <div>
          <span>SCORE:</span>
          <span>{{score}}</span>
        </div>
        <div>
          <span>LEVEL:</span>
          <span>{{level}}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
// import _ from 'lodash'

const ARROWUP = "ArrowUp";
const ARROWRIGHT = "ArrowRight";
const ARROWDOWN = "ArrowDown";
const ARROWLEFT = "ArrowLeft";
const BASESIZE = 20;  // 默认单位大小，px
const BASESCORE = 2; // 单个食物得分
const UPLEVELNUM = 8; // 获取多少食物后升级

@Component({})
export default class Home extends Vue {
  mounted(): void {
    // 添加监听键盘事件
    this.initGame();
    this._initProgram()
  }
  _initProgram():void {
    this.forbidenDirection = ''
    // 初始化游戏
    this.initGameArea();
    // 初始化蛇
    this.initSnake();
    // 初始化食物
    this.initFood();
    // 初始化计分板
    this.initScore();
  }

  // 游戏控制
  offsetWidth = 0;
  offsetHeight = 0;
  initGameArea(): void {
    let gameHtml = this.$refs.game as HTMLElement;
    this.offsetWidth = gameHtml.offsetWidth;
    this.offsetHeight = gameHtml.offsetHeight;
  }
  forbidenDirection = "";
  initGame(): void {
    document.addEventListener("keydown", (e) => {
      if (e.key === this.forbidenDirection) {
        this.$message.warning("亲，不能掉头哦！！！");
        return;
      }
      let x = 0;
      let y = 0;
      let flag = false;
      switch (e.key) {
        case ARROWUP:
          this.forbidenDirection = ARROWDOWN;
          y -= BASESIZE;
          flag = true;
          break;
        case ARROWRIGHT:
          this.forbidenDirection = ARROWLEFT;
          x += BASESIZE;
          flag = true;
          break;
        case ARROWDOWN:
          this.forbidenDirection = ARROWUP;
          y += BASESIZE;
          flag = true;
          break;
        case ARROWLEFT:
          this.forbidenDirection = ARROWRIGHT;
          x -= BASESIZE;
          flag = true;
          break;
      }
      if (flag) {
        this.snakeMove(x, y);
      }
    });
  }
  // 蛇
  snake: number[][] = [];
  time = 0;
  speed = 500
  initSnake(): void {
    this.snake = [[0, 0]];
    this.speed = 500
  }
  snakeMove(x: number, y: number): void {
    if (this.time) {
      clearTimeout(this.time);
    }
    let snakeHead = this.snake[0];
    let newSnakeHead = [snakeHead[0] + x, snakeHead[1] + y];
    if (
      newSnakeHead[0] < 0 ||
      newSnakeHead[0] > this.offsetWidth ||
      newSnakeHead[1] < 0 ||
      newSnakeHead[1] > this.offsetHeight
    ) {
      this.$message.error("啊哦，你撞墙了！！！");
      this._initProgram()
      return;
    }
    if (this.checkRepeat(newSnakeHead[0], newSnakeHead[1])) {
      this.$message.error("啊哦，你撞到自己了！！！");
      this._initProgram()
      return;
    }
    this.snake.unshift(newSnakeHead);
    if (newSnakeHead[0] === this.foodX && newSnakeHead[1] === this.foodY) {
      this.createFood();
      if(!this.addScore()) {
        this.$message.success("恭喜，您已通关!!!");
        this._initProgram()
        return;
      }
    } else {
      this.snake.pop();
    }
    this.time = setTimeout(() => this.snakeMove(x, y), this.speed);
  }
  checkRepeat(x: number, y: number): boolean {
    let repeat = this.snake.find((item) => item[0] === x && item[1] === y);
    return repeat ? true : false;
  }
  // 食物
  foodX = 0;
  foodY = 0;
  initFood(): void {
    this.createFood();
  }
  createFood(): void {
    this.foodX =
      Math.round(Math.random() * Math.floor(this.offsetWidth / BASESIZE)) *
      BASESIZE;
    this.foodY =
      Math.round(Math.random() * Math.floor(this.offsetHeight / BASESIZE)) *
      BASESIZE;
    if (this.checkRepeat(this.foodX, this.foodY)) {
      this.createFood();
    }
  }
  // 计分板：分数和等级
  score = 0
  level = 1
  initScore():void {
    this.score = 0
    this.level = 1
  }
  addScore():boolean {
    this.score += BASESCORE
    let foodNum = this.score / BASESCORE
    if (foodNum >= UPLEVELNUM){
      if(foodNum / UPLEVELNUM >= 3 ){
        return false
      }
      if(foodNum == UPLEVELNUM || foodNum == UPLEVELNUM * 2){
        this.$message.success("恭喜您，升级啦！");
        this.level += 1
        this.speed -= 100
      }

    }
    return true
  }
}
</script>
<style lang="scss" scoped>
.main {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100%;
  .main-title {
    font-size: 20px;
    font-weight: 600;
  }
  .main-body {
    width: 90%;
    height: 90%;
    background-color: #b7d4a8;
    border: 10px solid #000;
    border-radius: 10px;
    .game {
      width: 100%;
      height: 90%;
      border-bottom: 2px solid #000;
      position: relative;
      .snake {
        :first-child {
          background: url("~@/assets/imgs/snake.jpeg") no-repeat center;
          background-size: 300%;
        }
        > div {
          width: 20px;
          height: 20px;
          background: rgb(73, 176, 163);
          border: 1px solid #b7d4a8;
          box-sizing: border-box;
          position: absolute;
        }
      }
      .food {
        background-color: rgb(226, 40, 27);
        width: 20px;
        height: 20px;
        position: absolute;
      }
    }
    .score {
      width: 100%;
      height: 10%;
      display: flex;
      justify-content: space-around;
      align-items: center;
      font-size: 16px;
    }
  }
}
</style>