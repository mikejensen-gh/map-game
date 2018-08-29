<!-- https://medium.com/@immattjenkins/google-maps-in-plain-vue-6e0fc755f7d0 -->
<template>
  <div>
    <div id="gameInterface">
      <div v-if="gameState.status === 'inactive'" @click="startGame">Start game!</div>
      <div v-if="gameState.status === 'active'">
        <div>Cities found: {{ gameState.citiesCorrectlyGuessed }}</div>
        <div>Kilometers left: {{ gameState.kilometersLeft }}</div>
        <div>Find {{ gameState.cityToGuess.name }}</div>
      </div>
      <div v-if="gameState.status === 'gameOver'">
        Game Over! You found {{ gameState.citiesCorrectlyGuessed }} {{ gameState.citiesCorrectlyGuessed === 1 ? 'city' : 'cities'  }}, good job.
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
      
      gameState: {
        kilometersLeft: 1500,
        citiesCorrectlyGuessed: 0,
        status: 'inactive',
        cityToGuess: null
      },

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
        center: {lat: 52.37733, lng: 9.7304913},
        zoom: 5,

        mapTypeId: 'hybrid',
        
        fullscreenControl: false,
        mapTypeControl: false,
        streetViewControl: false,

        styles: [
          {
            featureType: 'administrative',
            elementType: 'labels',
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
        this.guessCityLocation(e);
      });
    },

    googleMapsFailedToLoad() {
      this.vueGMap = 'Error occurred';
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
      this.gameState.status = 'active';
      this.gameState.cityToGuess = this.cities[Math.floor(Math.random() * this.cities.length)];
    },

    guessCityLocation(event) {
      if (this.gameState.status !== 'active') {
        return;
      }

      const target = this.gameState.cityToGuess;

      const guess = {
        lat: event.latLng.lat(),
        lng: event.latLng.lng()
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
        this.gameState.cityToGuess = this.cities[Math.floor(Math.random() * this.cities.length)];
      } else {
        this.gameState.status = 'gameOver';
      }
    }
  },

  mounted() {
    // Source: https://medium.com/@immattjenkins/google-maps-in-plain-vue-6e0fc755f7d0

    this.createGoogleMaps().then(this.initGoogleMaps, this.googleMapsFailedToLoad)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
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
