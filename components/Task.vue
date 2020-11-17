<template>
  <section class="a-section categories-section">
    <div class="task" v-if="task">
      <h1>{{task.fields.name}}</h1>
      <div class="video">
        <iframe width="560" height="315" :src="task.fields.cartoonUrl" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
      </div>
      <div class="preview">
        <img :src="task.fields.preview.fields.file.url" alt="" width="560">
      </div>
      <div class="riddles">
        <div class="riddle" v-for="riddle in task.fields.riddles">
          <h1 class="question">{{riddle.fields.question}}</h1>
          <div class="answer">
            <img :src="riddle.fields.answer.fields.file.url" alt="" width="150">
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
<script>
  import { createClient } from '../plugins/contentful.js';
  const client = createClient();

  export default {
    data() {
      return {
        task: null,
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
          console.log(this.task);
        }
        catch (err) {
          console.log(err);
        }
      }
    },
    mounted() {
      this.getTask();
    }
  };
</script>

<style>

</style>
