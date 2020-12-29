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
      async getCategoryTasks(type) {
        try {
          const categoryTasks = await client.getEntries({
            'content_type': 'task',
          });
          this.categoryTasks = categoryTasks.items.filter(item => {
            return item.fields.tackType.fields.slug === type;
          });
          console.log(this.categoryTasks);
        }
        catch (err) {
          console.log(err);
        }
      }
    },
    mounted() {
      this.getCategoryTasks(this.$route.params.slug);
    }
  };
</script>

<style>

</style>
