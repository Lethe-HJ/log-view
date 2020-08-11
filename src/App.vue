<template>
  <div id="app">
    <HelloLogView />
  </div>
</template>

<script>
import HelloLogView from './components/HelloLogView.vue'

export default {
  name: 'App',
  components: {
    HelloLogView
  }
}
</script>

<style>

* {
  margin: 0;
  padding: 0;
}

body {
  font-size: 14px;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
}
</style>
