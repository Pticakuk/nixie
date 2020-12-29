<template>
  <div class="container">
    <h1 class="p-title">Выбери категорию</h1>
    <div>
      <Categories/>
    </div>
  </div>
</template>

<script>
  import Categories from './Categories';
  export default {
    comments: {
      Categories
    }
  }
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: #F7BBCF;
}
  .p-title {
    margin-bottom: 30px;
  }
</style>
