<template>
  <section class="a-section categories-section">
    <div class="task" v-if="task">
      <h1>{{task.fields.name}}</h1>
      <div class="video"
        :class="{'video--show': showCartoon}">
        <iframe width="560" height="315" :src="task.fields.cartoonUrl" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen>
        </iframe>
        <div class="puzzle-wrap">
          <div class="puzzle" v-for="puzzle in puzzleCount" :key="puzzle"></div>
        </div>
      </div>
      <div class="preview">
<!--        <img :src="task.fields.preview.fields.file.url" alt="" width="560">-->
      </div>
      <div class="riddles-wrap"
           :class="{'riddles-wrap--hide': showCartoon || hideCartoon}">
        <div content="active-riddle" v-if="activeRiddle">
          <h1 class="active-riddle__question">{{activeRiddle.fields.question}}</h1>
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
        puzzleCount: 10,
        rightAnsver: 0,
        attempts: 8,
        allRiddles: [],
        shovedRiddle: [],
        showCartoon: false,
        hideCartoon: false,
      }
    },
    computed: {
      activeRiddle() {
        return this.allRiddles.find(riddle => riddle.active);
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
          }
        } else if(this.attempts === 0){
          this.hideCartoon = true;
        }
        this.showRiddles();
        console.log(riddle);
      },
      randomInteger(min, max) {
        // случайное число от min до (max+1)
        let rand = min + Math.random() * (max + 1 - min);
        return Math.floor(rand);
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
    transition: 2000ms;
  }
  .riddle:hover {
    box-shadow: 0 0 10px blue;
  }
  .riddle img{
    position: absolute;
    height: 100%;
    width: auto;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
  }
  .active-riddle__question {
    text-align: center;
    font-size: 150px;
    color: #555;
    text-transform: uppercase;
    font-family: Arial, sans-serif;
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
    width: 20%;
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
</style>
