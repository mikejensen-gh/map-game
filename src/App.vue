<template>
  <div id="app">
      <v-app>
        <v-content>
          <MapGame/>
        </v-content>
      </v-app>
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
  font-size: 14px;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
}
</style>
