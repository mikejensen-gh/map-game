<template>
  <div>
    <div id="gameInterface">{{counter}}</div>
    <div id="gmap-container"></div>
  </div>
</template>

<script>
export default {
  name: 'MapGame',

  data() {
    return {
      vueGMap: null,
      counter: 0,

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
        zoom: 6,

        mapTypeId: 'terrain',
        
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

      this.vueGMap.addListener('click', (e) => {
        this.calculateSeparation(e);
      });
    },

    googleMapsFailedToLoad() {
      this.vueGMap = 'Error occurred';
    },

    calculateSeparation(event) {
      const target = {
        lat: 47.22,
        lng: 8.33,
      }

      const guess = {
        lat: event.latLng.lat(),
        lng: event.latLng.lng()
      }

      try {
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
        const separationDistanceInKm = earthRadiusInKm * angularDistance;

        console.log(separationDistanceInKm);
      } catch (e) {
        debugger;
      }
    }
    
  },

  mounted() {
    this.createGoogleMaps().then(this.initGoogleMaps, this.googleMapsFailedToLoad)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
body {
  background: #20262E;
  padding: 20px;
  font-family: Helvetica;
}

#app {
  background: #fff;
  border-radius: 4px;
  padding: 20px;
  transition: all 0.2s;
}

li {
  margin: 8px 0;
}

h2 {
  font-weight: bold;
  margin-bottom: 15px;
}

del {
  color: rgba(0, 0, 0, 0.3);
}

#gmap-container {
  height: 100vh;
}
</style>
