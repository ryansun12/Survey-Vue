<template>
  <div id="app">
    <survey> </survey>
  </div>
</template>

<script>
import survey from './components/survey.vue'
export default {
    name:'app',
    components: {
    survey
    }
  }

</script>
<style>
#app {
  font-family: Arial,'Avenir', Helvetica, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  font-size: 20px;
}
</style>
