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
  background: #20262E;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  text-align: center;

  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
}
</style>
