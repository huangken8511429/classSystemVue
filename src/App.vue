
<template>
  <div id="app">
    <ClassroomSchedule />
  </div>
</template>

<script>
import ClassroomSchedule from './components/ClassroomSchedule.vue'

export default {
  name: 'App',
  components: {
    ClassroomSchedule
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
