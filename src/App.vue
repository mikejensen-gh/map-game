<template>
<div id="app">
    <v-app>
      <v-content>
        <GameInstructions
          :gameActive="gameActive"
          :gameOver="gameOver"
          :gameState="gameState"

          @startGame="startGame()"
        ></GameInstructions>

        <GuessModal
          :confirmGuessText="confirmGuessText"
          :showGuessModal="showGuessModal"
          :guessConfirmed="guessConfirmed"
          :gameActive="gameActive"
          :gameOver="gameOver"

          @confirmGuess="confirmGuess()"
          @nextRound="nextRound()"
          @startGame="startGame()"
        ></GuessModal>

        <Map
          @clickHandler="mapClickHandler($event)"
          @setMap="setMap($event)"
        ></Map>
      </v-content>
    </v-app>
</div>
</template>

<script>
/* global google */

import GameInstructions from './components/GameInstructions.vue'
import GuessModal from './components/GuessModal.vue'
import Map from './components/Map.vue'

const cities = [
  {
    name: "Amsterdam",
    lat: 52.3546274,
    lng: 4.828584
  },
  {
    name: "Athens",
    lat: 37.990832,
    lng: 23.70332
  },
  {
    name: "Belgrade",
    lat: 44.8151597,
    lng: 20.2825159
  },
  {
    name: "Berlin",
    lat: 52.5067614,
    lng: 13.2846515
  },
  {
    name: "Bratislava",
    lat: 48.1356952,
    lng: 16.9758363
  },
  {
    name: "Brussels",
    lat: 50.8549541,
    lng: 4.3053509
  },
  {
    name: "Bucharest",
    lat: 44.4378043,
    lng: 26.0245983
  },
  {
    name: "Budapest",
    lat: 47.4811282,
    lng: 18.9902222
  },
  {
    name: "Chisinau",
    lat: 46.9998477,
    lng: 28.7881374
  },
  {
    name: "Copenhagen",
    lat: 55.6712474,
    lng: 12.523785
  },
  {
    name: "Dublin",
    lat: 53.3242996,
    lng: -6.3157431
  },
  {
    name: "Helsinki",
    lat: 60.1098679,
    lng: 24.738515
  },
  {
    name: "Istanbul",
    lat: 41.0052367,
    lng: 28.872098
  },
  {
    name: "Lisabon",
    lat: 38.7436214,
    lng: -9.1952225
  },
  {
    name: "Ljubljana",
    lat: 46.066096,
    lng: 14.4620599
  },
  {
    name: "London",
    lat: 51.5285582,
    lng: -0.241679
  },
  {
    name: "Kyiv",
    lat: 50.4019514,
    lng: 30.39261
  },
  {
    name: "Madrid",
    lat: 40.4379332,
    lng: -3.749576
  },
  {
    name: "Minsk",
    lat: 53.8838069,
    lng: 27.5796489
  },
  {
    name: "Moscow",
    lat: 55.5807482,
    lng: 36.8251602
  },
  {
    name: "Oslo",
    lat: 59.8938364,
    lng: 10.7150777
  },
  {
    name: "Paris",
    lat: 48.8588536,
    lng: 2.3120407
  },
  {
    name: "Podgorica",
    lat: 42.4319954,
    lng: 19.2208596
  },
  {
    name: "Prague",
    lat: 50.0595854,
    lng: 14.3255434
  },
  {
    name: "Prishtina",
    lat: 42.6663748,
    lng: 21.123708
  },
  {
    name: "Riga",
    lat: 56.9713962,
    lng: 23.9890827
  },
  {
    name: "Rome",
    lat: 41.909986,
    lng: 12.3959165
  },
  {
    name: "Sarajevo",
    lat: 43.8937019,
    lng: 18.3129523
  },
  {
    name: "Skopje",
    lat: 41.9990849,
    lng: 21.389871
  },
  {
    name: "Sofia",
    lat: 42.6388078,
    lng: 23.1838654
  },
  {
    name: "Stockholm",
    lat: 59.3260668,
    lng: 17.841973
  },
  {
    name: "Tallinn",
    lat: 59.4716181,
    lng: 24.598162
  },
  {
    name: "Tirana",
    lat: 41.3309769,
    lng: 19.7828039
  },
  {
    name: "Vienna",
    lat: 48.2206636,
    lng: 16.3100208
  },
  {
    name: "Vilnius",
    lat: 54.7000902,
    lng: 25.1128521
  },
  {
    name: "Warsaw",
    lat: 52.232855,
    lng: 20.9211138
  },
  {
    name: "Zagreb",
    lat: 45.8401746,
    lng: 15.8942924
  },
  {
    name: "Zurich",
    lat: 47.3774497,
    lng: 8.5016958
  },
];

export default {
  name: 'app',

  components: {
    GameInstructions,
    GuessModal,
    Map
  },

  data() {
    return {
      vueGMap: null,

      gameActive: false,
      gameOver: false,

      gameState: {
        kilometersLeft: 1500,
        citiesCorrectlyGuessed: 0,
        cityToGuess: {
          name: '',
          lat: 0,
          lng: 0
        }
      },

      confirmGuessText: null,
      guessConfirmed: false,
      showGuessModal: false,

      guessMarker: null,
      targetMarker: null,
    }
  },

  methods: {
    startGame() {
      this.gameState = {
        kilometersLeft: 1500,
        citiesCorrectlyGuessed: 0,
        cityToGuess: {
          name: '',
          lat: 0,
          lng: 0
        }
      }

      this.gameActive = true;
      this.gameOver = false;
      this.nextRound();
    },

    showConfirmGuessModal(event) {
      if (this.guessMarker) {
        this.guessMarker.setMap(null);
      }

      this.confirmGuessText = 'Confirm guess?';
      this.showGuessModal = true;

      this.guessMarker = new google.maps.Marker({
        position: event.latLng,
        map: this.vueGMap,
      })
    },

    confirmGuess() {
      if (!this.gameActive) {
        return;
      }

      const target = this.gameState.cityToGuess;

      const guess = {
        lat: this.guessMarker.position.lat(),
        lng: this.guessMarker.position.lng()
      }

      const separationDistanceInKm = this.getGuessDistance(target, guess);

      if (separationDistanceInKm === null) {
        return;
      }

      if (separationDistanceInKm <= 50) {
        this.gameState.citiesCorrectlyGuessed++;
        this.confirmGuessText = `Excellent! Your guess was ${separationDistanceInKm}km from ${this.gameState.cityToGuess.name}`;
      } else {
        this.gameState.kilometersLeft -= separationDistanceInKm;
        this.confirmGuessText = `Oops, your guess was ${separationDistanceInKm}km from ${this.gameState.cityToGuess.name}`;
        this.targetMarker = new google.maps.Marker({
          position: {
            lat: target.lat,
            lng: target.lng,
          },
          map: this.vueGMap,
          animation: google.maps.Animation.BOUNCE
        });

        this.vueGMap.panTo(this.targetMarker.getPosition());

        setTimeout(() => {
          this.targetMarker.setAnimation(null);
        }, 1000);
      }

      this.guessConfirmed = true;

      if (this.gameState.kilometersLeft <= 0) {
        this.gameOver = true;
      }
    },

    toRadians(valueInDegrees) {
      return valueInDegrees * (Math.PI / 180)
    },

    getGuessDistance(target, guess) {
      try {
        // Source: http://www.movable-type.co.uk/scripts/latlong.html

        const earthRadiusInKm = 6371;
        const targetLatInRadians = this.toRadians(target.lat);
        const guessLatInRadians = this.toRadians(guess.lat);
        const latDiffInRadians = this.toRadians(guess.lat-target.lat);
        const lngDiffInRadians = this.toRadians(target.lng-guess.lng);

        const squareOfHalfOfChordLengthBetweenPoints = (Math.sin(latDiffInRadians/2) * Math.sin(latDiffInRadians/2))
          + (Math.cos(targetLatInRadians) * Math.cos(guessLatInRadians) * Math.sin(lngDiffInRadians/2) * Math.sin(lngDiffInRadians/2));
        const angularDistance = 2 * Math.atan2(Math.sqrt(squareOfHalfOfChordLengthBetweenPoints), Math.sqrt(1-squareOfHalfOfChordLengthBetweenPoints));

        return Math.round(earthRadiusInKm * angularDistance); // Distance in KM
      } catch (e) {
        console.error('Error calculating distance between two points', e);
        return null;
      }
    },

    nextRound() {
      if (this.guessMarker) {
        this.guessMarker.setMap(null);
      }

      if (this.targetMarker) {
        this.targetMarker.setMap(null);
      }

      this.guessConfirmed = false;
      this.showGuessModal = false;
      this.selectNextCity();
    },

    selectNextCity() {
      const previousCity = this.gameState.cityToGuess;

      do {
        this.gameState.cityToGuess = cities[Math.floor(Math.random() * cities.length)];
      } while (previousCity && this.gameState.cityToGuess.name === previousCity.name)
    },

    mapClickHandler(event) {
      if (this.guessConfirmed) {
        return;
      }

      this.showConfirmGuessModal(event);
    },

    setMap(map) {
      this.vueGMap = map;
    }
  },
}
</script>

<style scoped>
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
