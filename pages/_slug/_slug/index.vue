<template>
  <div class="container">
      <Task/>
  </div>
</template>

<script>
  import Task from '../../../components/Task';
export default {
  comments: {
    Task
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
</style>
