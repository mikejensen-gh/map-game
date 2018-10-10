<template>
  <v-card id="instructions">
    <v-card-title primary-title>
      <div>
        <h3 class="headline mb-2" v-html="headingText"></h3>
        <div class="subheading" v-html="instructionText"></div>
      </div>
    </v-card-title>
    <v-card-actions>
      <v-btn v-if="!gameActive" @click="$emit('startGame')" class="mx-auto" color="primary">Start game!</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
export default {
  name: 'GameInstructions',

  props: [
    'gameActive',
    'gameOver',
    'gameState',
    'startGame'
  ],

  computed: {
    instructionText: function() {
      let message = '';
      const score = this.gameState.citiesCorrectlyGuessed;

      if (!this.gameActive) {
        message = `
         Try to find the cities - guesses <b>50km or closer</b> are counted as correct. Otherwise you'll lose points; when your score hits 0, the game is over. Good luck!
        `;
      } else if (this.gameOver) {
        message = `You found ${score} ${score === 1 ? 'city' : 'cities'  }, `

        switch (true) {
          case score <= 3: {
            message += 'better luck next time.';
            break;
          }

          case score <= 10: {
            message += 'not half bad!'
            break;
          }

          default: {
            message += 'nice work!'
          }
        }
      } else {
        message = `
          Kilometers left: <b>${this.gameState.kilometersLeft}</b><br>
          Cities found: <b>${this.gameState.citiesCorrectlyGuessed}</b><br>
        `
      }

      return message;

    },

    headingText: function() {
      if (this.gameOver) {
        return 'Game over!'
      } else if (this.gameState.cityToGuess) {
        return `Can you find... <b>${this.gameState.cityToGuess.name}</b>?`
      } else {
        return 'Test your knowledge!'
      }
    }
  }
}
</script>

<style scoped>
#instructions {
  position: fixed;
  z-index: 1;
  margin: 5px auto 0;
  left: 0;
  right: 0;

  max-width: 500px;
}
</style>
