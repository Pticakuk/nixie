<template>
  <section class="a-section categories-section">
    <div class="task" v-if="task">
      <h1>{{task.fields.name}}</h1>
      <div class="video"
        :class="{'video--show': showCartoon}">
        <iframe width="560" height="315" :src="task.fields.cartoonUrl" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen>
        </iframe>
        <div class="puzzle-wrap">
          <div class="puzzle" v-for="puzzle in puzzleCount" :key="puzzle"
          :style="`width: ${100 / (puzzleCount / 2)}%`"></div>
        </div>
      </div>
      <div class="preview">
<!--        <img :src="task.fields.preview.fields.file.url" alt="" width="560">-->
      </div>
      <div class="riddles-wrap"
           :class="{'riddles-wrap--hide': showCartoon || hideCartoon}">
        <div v-if="tackType === 'reading'">
          <div class="active-riddle" v-if="activeRiddle">
            <div v-if="activeRiddle.fields.riddle && activeRiddle.fields.riddle.content && activeRiddle.fields.riddle.content.length"
            class="active-riddle__text">
              <p v-for="p in activeRiddle.fields.riddle.content">
                {{p.content[0].value}}
              </p>
            </div>
            <h1 class="active-riddle__question" v-else>{{activeRiddle.fields.question}}</h1>

          </div>
          <div class="riddles">
            <div class="riddle" v-for="riddle in shovedRiddle">
              <div class="answer"
                   @click="selectAnswer(riddle)">
                <img :src="riddle.fields.answer.fields.file.url" alt="" width="150">
              </div>
            </div>
          </div>
        </div>
        <div v-if="tackType === 'mathematics'">
          <div class="active-math-riddle" v-if="activeRiddle">
            <div class="math-riddle__number">{{activeRiddle.fields.firstNumber}}</div>
            <div class="math-riddle__sign">{{activeRiddle.fields.sign}}</div>
            <div class="math-riddle__number">{{activeRiddle.fields.secondNumber}}</div>
          </div>
          <div class="math-riddle__answer">
            <input type="number" ref="mathAnswer" v-model="mathAnsver">
            <button @click="onMathAnswer(activeRiddle)">Вiдповicти</button>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
<script>
  import { createClient } from '../plugins/contentful.js';
  import JQuery from 'jquery'
  let $ = JQuery;
  const client = createClient();

  export default {
    data() {
      return {
        task: null,
        puzzleCount: 4,
        rightAnsver: 0,
        attempts: 4,
        allRiddles: [],
        shovedRiddle: [],
        showCartoon: false,
        hideCartoon: false,
        mathAnsver: '',
      }
    },
    computed: {
      activeRiddle() {
        console.log(this.allRiddles.find(riddle => riddle.active));
        return this.allRiddles.find(riddle => riddle.active);
      },
      tackType() {
        return this.task.fields.tackType.fields.slug;
      }
    },
    methods: {
      async getTask() {
        try {
          const task = await client.getEntries({
            'content_type': 'task',
            'fields.slug[in]': this.$route.params.slug,
          });
          this.task = task.items[0];
          this.allRiddles = this.task.fields.riddles
            .map((riddle) => {
              return {
                ...riddle,
                active: false,
                alreadyShown: false,
              }
            })
            .sort(() => .5 - Math.random());
          this.attempts = this.allRiddles.length;
          this.showRiddles();
        }
        catch (err) {
          console.log(err);
        }
      },
      showRiddles() {
        this.allRiddles = this.allRiddles.map((riddle) => {
          return {...riddle, active: false}
        });
        const currentRiddle = this.allRiddles.find(riddle => !riddle.alreadyShown);
        currentRiddle.active = true;
        currentRiddle.alreadyShown = true;
        this.shovedRiddle = [...this.allRiddles.filter(riddle => !riddle.active).slice(0,7), currentRiddle];
        this.allRiddles.sort(() => .5 - Math.random());
        this.shovedRiddle.sort(() => .5 - Math.random());
      },
      selectAnswer(riddle) {
        this.attempts -= 1;
        if (this.activeRiddle.sys.id === riddle.sys.id) {
          this.rightAnsver += 1;
          const puzzleNumber = (this.randomInteger(0, $('.puzzle').length));
          $($('.puzzle')[0]).removeClass('puzzle').addClass('puzzle--hidden');
          if (this.rightAnsver >= this.puzzleCount) {
            this.puzzleCount -= 1;
            this.showCartoon = true;
            return;
          }
        } else if(this.attempts === 0){
          this.hideCartoon = true;
          return;
        }
        this.showRiddles();
      },
      randomInteger(min, max) {
        // случайное число от min до (max+1)
        let rand = min + Math.random() * (max + 1 - min);
        return Math.floor(rand);
      },
      onMathAnswer(mathRiddle) {
        this.attempts -= 1;
        if (this.mathAnsver) {
          const sign = mathRiddle.fields.sign;
          let answer = '';
          switch (sign) {
            case '-': {
              answer = mathRiddle.fields.firstNumber - mathRiddle.fields.secondNumber;
              break;
            }
            case '*': {
              answer = mathRiddle.fields.firstNumber * mathRiddle.fields.secondNumber;
              break;
            }
            case '/': {
              answer = mathRiddle.fields.firstNumber / mathRiddle.fields.secondNumber;
              break;
            }
            default: {
              answer = mathRiddle.fields.firstNumber + mathRiddle.fields.secondNumber;
            }
          }
          if(answer === (this.mathAnsver)*1) {
            this.rightAnsver += 1;
            const puzzleNumber = (this.randomInteger(0, $('.puzzle').length));
            $($('.puzzle')[0]).removeClass('puzzle').addClass('puzzle--hidden');
            if (this.rightAnsver >= this.puzzleCount) {
              this.puzzleCount -= 1;
              this.showCartoon = true;
            }
          } else if(this.attempts === 0){
            this.hideCartoon = true;
            return;
          }
            this.mathAnsver = '';
            this.$refs.mathAnswer.focus()
            this.showRiddles();
          }
      }
    },
    mounted() {
      this.getTask();
    }
  };
</script>

<style>
  .riddles {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
    max-width: 1000px;
    margin: 0 auto;
  }
  .riddle {
    width: 200px;
    height: 200px;
    cursor: pointer;
    position: relative;
    margin: 20px;
    overflow: hidden;
    transition: 500ms;
  }
  .riddle:hover {
    box-shadow: 0 0 10px brown;
  }
  .riddle img{
    position: absolute;
    height: auto;
    width: 100%;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
  }
  .active-riddle__question,
  .active-riddle__text{
    text-align: center;
    font-size: 150px;
    color: #555;
    font-family: Arial, sans-serif;
  }
  .active-riddle__text{
    font-size: 40px;
    font-weight: 900;
    white-space: pre-line;
  }
  .video {
    width: 560px;
    height: 315px;
    margin: 0 auto;
    position: relative;
  }
  .video iframe {
    position: relative;
    z-index: 1;
    transition: 2000ms;
  }
  .video--show .puzzle-wrap {
    opacity: 0;
    max-height: 0;
  }
  .video--show iframe {
    width: 700px;
    height: 400px;
  }
  .puzzle-wrap {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: stretch;
    justify-content: space-between;
    flex-wrap: wrap;
    position: absolute;
    z-index: 2;
    top:0;
    left:0;
  }
  .riddles-wrap {
    max-height: 1000px;
    transition: 2000ms;
  }
  .riddles-wrap--hide {
    opacity: 0;
    max-height: 0;
    overflow: hidden;
  }
  .puzzle,
  .puzzle--hidden {
    height: 50%;
    background: grey;
    opacity: 0.9;
    background: url("../assets/images/questions.jpg") center no-repeat rgba(255,255,255, 0.7);
    background-size: 100% 100%;
    transition: 500ms;
  }
  .puzzle--hidden {
    opacity: 0;
  }
  .active-math-riddle,
  .math-riddle__answer {
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: Arial, sans-serif;
    margin-top: 50px;
  }
  .math-riddle__number {
    font-size: 90px;
    font-family: Arial, sans-serif;
  }
  .math-riddle__sign {
    font-size: 100px;
    margin: 0 50px;
    font-weight: bold;
  }
  .math-riddle__answer {
    flex-direction: column;
  }
  .math-riddle__answer input {
    font-size: 70px;
    text-align: center;
    width: 300px;
    height: 60px;
    color: darkblue;
    border: 0;
    outline: none;
    box-shadow: 2px 2px 10px #777;
  }
  .math-riddle__answer button {
    background: rgb(2,147,113);
    color: white;
    font-size: 40px;
    border-radius: 5px;
    height: 60px;
    margin-top: 30px;
    border: 0;
    box-shadow: 2px 2px 10px #777;
    padding:  0 50px;
    transition: 300ms;
  }
  .math-riddle__answer button:hover {
    background: rgb(54,183,124);
  }
</style>
