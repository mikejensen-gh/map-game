<template>
<div id="gmap-container"></div>
</template>

<script>
const mapOptions = {
  center: {
    lat: 52.37733,
    lng: 9.7304913
  },
  zoom: 6,

  mapTypeId: 'terrain',
  gestureHandling: 'greedy',

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

export default {
  name: 'Map',

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
      const map = new google.maps.Map(document.getElementById('gmap-container'), mapOptions);

      map.addListener('click', (e) => {
        this.$emit('clickHandler', e);
      });

      this.$emit('setMap', map)
    },

    googleMapsFailedToLoad() {
      const map = 'Error occurred';

      this.$emit('setMap', map)
    },
  },

  mounted() {
    this.createGoogleMaps().then(this.initGoogleMaps, this.googleMapsFailedToLoad)
  },
}
</script>

<style scoped>
#gmap-container {
  height: 100vh;
}
</style>
