<template>
  <div>
    <v-card id="gameInterface">
      <v-layout>
        <v-flex>
          <v-card-title primary-title>
            <div>
              <h3 class="headline mb-2">{{ headingText }}</h3>
              <div class="subheading" v-html="gameInstructions()"></div>
            </div>
          </v-card-title>
          <v-card-actions>

            <v-btn v-if="!gameActive" @click="startGame" flat class="indigo mx-auto">Start game!</v-btn>

          </v-card-actions>
        </v-flex>
      </v-layout>
      <v-snackbar v-model="showGuessModal" bottom :timeout="0">
        {{ confirmGuessText }}

        <v-btn v-if="!guessConfirmed" color="pink" flat @click="confirmGuess">Confirm</v-btn>
        <v-btn v-if="guessConfirmed && gameActive && !gameOver" color="pink" flat @click="nextRound">Continue</v-btn>
        <v-btn v-if="gameOver" color="pink" flat @click="startGame">Start over</v-btn>

      </v-snackbar>
    </v-card>
    <div id="gmap-container"></div>
  </div>
</template>

<script>
export default {
  name: 'MapGame',

  data() {
    return {
      vueGMap: null,

      gameActive: false,
      gameOver: false,

      gameState: {},

      confirmGuessText: null,
      guessConfirmed: false,
      showGuessModal: false,

      guessMarker: null,
      targetMarker: null,

      cities: [
          {
              name: "Amsterdam",
              lat: 52.3546274,
              lng: 4.828584
          },
          {
              name: "Berlin",
              lat: 52.5067614,
              lng: 13.2846515
          },
          {
              name: "London",
              lat: 51.5285582,
              lng: -0.241679
          },
          {
              name: "Madrid",
              lat: 40.4379332,
              lng: -3.749576
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
              name: "Rome",
              lat: 41.909986,
              lng: 12.3959165
          },
          {
              name: "Vienna",
              lat: 48.2206636,
              lng: 16.3100208
          },
          {
              name: "Zurich",
              lat: 47.3774497,
              lng: 8.5016958
          },
      ],
    }
  },

  methods: {
    createGoogleMaps () {
      // Google Maps + Vue implementation source: https://medium.com/@immattjenkins/google-maps-in-plain-vue-6e0fc755f7d0

      return new Promise((resolve, reject) => {
        let gmap = document.createElement('script')
        gmap.src = `https://maps.googleapis.com/maps/api/js?key=${process.env.VUE_APP_GMAPS_API}`
        gmap.type = 'text/javascript'
        gmap.onload = resolve
        gmap.onerror = reject
        document.body.appendChild(gmap)
      })
    },

    initGoogleMaps() {
      const options = {
        center: {
          lat: 52.37733,
          lng: 9.7304913
        },
        zoom: 6,

        mapTypeId: 'terrain',

        fullscreenControl: false,
        mapTypeControl: false,
        streetViewControl: false,


        // https://developers.google.com/maps/documentation/javascript/examples/hiding-features
        styles: [
          {
            featureType: 'administrative',
            elementType: 'labels',
            stylers: [
              {visibility: 'off'}
            ]
          },
          {
            featureType: 'administrative.province',
            stylers: [
              {visibility: 'off'}
            ]
          },
          {
            featureType: 'administrative.locality',
            stylers: [
              {visibility: 'off'}
            ]
          },
          {
            featureType: 'administrative.neighborhood',
            stylers: [
              {visibility: 'off'}
            ]
          },
          {
            featureType: 'administrative.land_parcel',
            stylers: [
              {visibility: 'off'}
            ]
          },
          {
            featureType: 'road',
            stylers: [
              {visibility: 'off'}
            ]
          },
          {
            featureType: 'poi',
            stylers: [
              {visibility: 'off'}
            ]
          },
          {
            featureType: 'transit',
            stylers: [
              {visibility: 'off'}
            ]
          },
          {
            featureType: 'water',
            elementType: 'labels',
            stylers: [
              {visibility: 'off'}
            ]
          },
          {
            featureType: 'landscape',
            elementType: 'labels',
            stylers: [
              {visibility: 'off'}
            ]
          },
        ],
      };

      this.vueGMap = new google.maps.Map(document.getElementById('gmap-container'), options);

      this.vueGMap.controls[google.maps.ControlPosition.TOP_CENTER].push(document.getElementById('gameInterface'));

      this.vueGMap.addListener('click', (e) => {
        if (this.guessConfirmed) {
          return;
        }

        this.showConfirmGuessModal(e);
      });
    },

    googleMapsFailedToLoad() {
      this.vueGMap = 'Error occurred';
    },

    startGame() {
      this.gameState = {
        kilometersLeft: 1500,
        citiesCorrectlyGuessed: 0,
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
      }

      this.guessConfirmed = true;

      if (this.gameState.kilometersLeft <= 0) {
        this.gameOver = true;
      }
    },

    getGuessDistance(target, guess) {
      try {
        // Source: http://www.movable-type.co.uk/scripts/latlong.html

        function toRadians(valueInDegrees) {
          return valueInDegrees * (Math.PI / 180)
        }

        const earthRadiusInKm = 6371;
        const targetLatInRadians = toRadians(target.lat);
        const guessLatInRadians = toRadians(guess.lat);
        const latDiffInRadians = toRadians(guess.lat-target.lat);
        const lngDiffInRadians = toRadians(target.lng-guess.lng);

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
        this.gameState.cityToGuess = this.cities[Math.floor(Math.random() * this.cities.length)];
      } while (previousCity && this.gameState.cityToGuess.name === previousCity.name)
    },

    gameInstructions: function() {
      let message = '';
      const score = this.gameState.citiesCorrectlyGuessed;

      if (!this.gameActive) {
        message = `
          Can you find Europe's capital cities?<br>
          If your guess is more than 50km away, you'll lose points.<br>
          When your score hits 0, the game is over. Good luck!
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
          Cities found: <span class="font-weight-bold">${this.gameState.citiesCorrectlyGuessed}</span><br>
          Kilometers left: <span class="font-weight-bold">${this.gameState.kilometersLeft}</span><br>
          Find <span class="font-weight-bold">${this.gameState.cityToGuess.name}!</span>
        `
      }

      return message;
    },
  },

  mounted() {
    this.createGoogleMaps().then(this.initGoogleMaps, this.googleMapsFailedToLoad)
  },

  computed: {
    headingText: function() {
      if (this.gameOver) {
        return 'Game over!'
      } else {
        return 'Can you find the city?'
      }
    }
  }
}
</script>

<style scoped>
#gmap-container {
  height: 100vh;
}

#gameInterface {
  z-index: 1;
  margin-top: 10px;
}
</style>
