<template>
  <section class="a-section categories-section">
    <div class="category" v-for="categiry in categiries">
      <nuxt-link class="category-link" :to="`/${categiry.fields.slug}`">{{categiry.fields.name}}</nuxt-link>
    </div>
  </section>
</template>
<script>
  import { createClient } from '../plugins/contentful.js';
  const client = createClient();

  export default {
    data() {
      return {
        categiries: [],
      }
    },
    methods: {
      async getCategories() {
        try {
          const response = await client.getEntries({
            'content_type': 'taskType',
            select: 'sys.id,fields.slug,fields.active,fields.name',
          });
          this.categiries = response.items
        }
        catch (err) {
          console.log(err);
        }
      }
    },
    mounted() {
      this.getCategories();
    }
  };
</script>

<style>
.category a {
  font-size: 30px;
  color: white;
}
</style>
