<template>
  <div id="gmap-container"></div>
</template>

<script>
export default {
  name: 'MapGame',
  data() {
    return {
      vueGMap: null,
    }
  },
  methods: {
    createGoogleMaps () {
      return new Promise((resolve, reject) => {
        let gmap = document.createElement('script')
        gmap.src = 'https://maps.googleapis.com/maps/api/js?key=AIzaSyA5vzR_Ej7ccka5ERQeJgGj8Bs7CZ0q_EI'
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
    },
    googleMapsFailedToLoad() {
      this.vueGMap = 'Error occurred';
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
