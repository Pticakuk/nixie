<template>
  <section class="a-section categories-section">
    <div class="category" v-for="categiry in categoryTasks">
      <nuxt-link :to="`/${$route.params.slug}/${categiry.fields.slug}`">{{categiry.fields.name}}</nuxt-link>
    </div>
  </section>
</template>
<script>
  import { createClient } from '../plugins/contentful.js';
  const client = createClient();

  export default {
    data() {
      return {
        categoryTasks: [],
      }
    },
    methods: {
      async getCategoryTasks() {
        try {
          const categoryTasks = await client.getEntries({
            'content_type': 'task',
          });
          this.categoryTasks = categoryTasks.items
        }
        catch (err) {
          console.log(err);
        }
      }
    },
    mounted() {
      console.log(this.$route.params.slug);
      this.getCategoryTasks();
    }
  };
</script>

<style>

</style>
