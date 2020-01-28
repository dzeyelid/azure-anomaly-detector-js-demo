<template>
  <div id="app">
    <img alt="Vue logo" width="100" src="./assets/logo.png">
    <anomaly-detector-demo />
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import AnomalyDetectorDemo from './components/AnomalyDetectorDemo.vue'

export default Vue.extend({
  name: 'app',
  components: {
    AnomalyDetectorDemo
  }
})
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}
</style>
