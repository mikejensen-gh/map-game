<template>
    <v-snackbar id="guessModal" v-model="showGuessModal" bottom :timeout="0">
      {{ confirmGuessText }}

      <v-btn v-if="!guessConfirmed" color="pink" flat @click="$emit('confirmGuess')">Confirm</v-btn>
      <v-btn v-if="guessConfirmed && gameActive && !gameOver" color="pink" flat @click="$emit('nextRound')">Continue</v-btn>
      <v-btn v-if="gameOver" color="pink" flat @click="$emit('startGame')">Start over</v-btn>

    </v-snackbar>
</template>

<script>
export default {
  name: 'GuessModal',

  props: [
    'confirmGuessText',
    'showGuessModal',
    'guessConfirmed',
    'gameActive',
    'gameOver'
  ],

  methods: {
  }
}
</script>

<style scoped>
#guessModal {
  z-index: 1;
}
</style>
