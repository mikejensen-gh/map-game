<template>
  <div id="app">
    <MapGame/>
  </div>
</template>

<script>
import MapGame from './components/MapGame.vue'

export default {
  name: 'app',
  components: {
    MapGame
  }
}
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}
</style>
