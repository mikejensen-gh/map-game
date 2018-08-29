<template>
  <div>
    <div id="gameInterface">
      <div v-if="gameStatus === 'inactive'">
        <div><button  @click="startGame">Start game!</button></div>
      </div>
      <div v-if="gameStatus === 'active'">
        <div>Cities found: {{ gameState.citiesCorrectlyGuessed }}</div>
        <div>Kilometers left: {{ gameState.kilometersLeft }}</div>
        <div>Find {{ gameState.cityToGuess.name }}</div>
        <div><button v-if="mapMarker !== null" @click="guessCityLocation">Confirm guess?</button></div>
      </div>
      <div v-if="gameStatus === 'gameOver'">
        <div>{{ gameOverMessage }}</div>
        <div><button @click="startGame">Click to restart</button></div>
      </div>
    </div>
    <div id="gmap-container"></div>
  </div>
</template>

<script>
export default {
  name: 'MapGame',

  data() {
    return {
      vueGMap: null,
      
      gameStatus: 'inactive',

      gameState: {},

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

      mapMarker: null,
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
        this.addMapMarker(e);
      });
    },

    googleMapsFailedToLoad() {
      this.vueGMap = 'Error occurred';
    },

    addMapMarker(event) {
      if (this.mapMarker) {
        this.mapMarker.setMap(null);
      }

      this.mapMarker = new google.maps.Marker({
        position: event.latLng,
        map: this.vueGMap,
      })
    },

    calculateSeparation(target, guess) {
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

    startGame() {
      this.gameState = {
        kilometersLeft: 1500,
        citiesCorrectlyGuessed: 0,
      }

      this.gameStatus = 'active';
      this.selectNextCity();
    },

    selectNextCity() {
      const previousCity = this.gameState.cityToGuess;

      do {
        this.gameState.cityToGuess = this.cities[Math.floor(Math.random() * this.cities.length)];
      } while (previousCity && this.gameState.cityToGuess.name === previousCity.name)
    },

    guessCityLocation() {
      if (this.gameStatus !== 'active') {
        return;
      }

      const target = this.gameState.cityToGuess;

      const guess = {
        lat: this.mapMarker.position.lat(),
        lng: this.mapMarker.position.lng()
      }

      const separationDistanceInKm = this.calculateSeparation(target, guess);

      if (separationDistanceInKm === null) {
        return;
      }

      if (separationDistanceInKm <= 50) {
        this.gameState.citiesCorrectlyGuessed++;
      } else {
        this.gameState.kilometersLeft -= separationDistanceInKm;
      }

      if (this.gameState.kilometersLeft > 0) {
        this.selectNextCity();
      } else {
        this.gameStatus = 'gameOver';
      }
    }
  },

  mounted() {
    this.createGoogleMaps().then(this.initGoogleMaps, this.googleMapsFailedToLoad)
  },

  computed: {
    gameOverMessage: function() {
      const score = this.gameState.citiesCorrectlyGuessed;
      let message = `Game Over! You found ${score} ${score === 1 ? 'city' : 'cities'  }, `

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

      return message;
    }
  }
}
</script>

<style scoped>
#gmap-container {
  height: 100vh;
}

#gameInterface {
  background: #FFF;
  padding: 10px;
  margin-top: 5px;
  box-shadow: 0px 0px 2px rgba(0,0,0,0.2);
}
</style>
