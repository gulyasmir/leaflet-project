<template>
  <div class="map">
    <LeafletMapComponent :citiesList="cities" :mapsList="maps" />
  </div>
</template>

<script>
import forecastStations from '../static/json/forecast_stations.json'
import maps from '../static/json/maps.json'
export default {
  name: "IndexPage",

  data() {
    return {
      forecastStations: forecastStations,
      cities: [
        {
          id: 1,
          title: "Москва",
          coords: {
            latitude: 55.7522,
            longitude: 37.6156,
          },
          forecasts: {
            forecastsIcon: "sun.svg",
            temperature: -1,
            min: 6,
            max: 15,
          },
          wind: {
            iconRotate: 20,
            blackWindSpeed: 20,
            redWindSpeed: 45,
          },
        },
        {
          id: 2,
          title: "Астрахань",
          coords: {
            latitude: 46.3497,
            longitude: 48.0408,
          },
          forecasts: {
            forecastsIcon: "rain.svg",
            temperature: 17,
            min: 6,
            max: 15,
          },
          wind: {
            iconRotate: 190,
            blackWindSpeed: 20,
            redWindSpeed: 45,
          },
        },
        {
          id: 3,
          title: "Барнаул",
          coords: {
            latitude: 53.3606,
            longitude: 83.7636,
          },
          forecasts: {
            forecastsIcon: "sunrain.svg",
            temperature: 10,
            min: 6,
            max: 15,
          },
          wind: {
            iconRotate: 40,
            blackWindSpeed: 20,
            redWindSpeed: 45,
          },
        },
        {
          id: 4,
          title: "Брянск",
          coords: {
            latitude: 53.2521,
            longitude: 34.3717,
          },
          forecasts: {
            forecastsIcon: "storm.svg",
            temperature: 5,
            min: 6,
            max: 15,
          },
          wind: {
            iconRotate: 70,
            blackWindSpeed: 20,
            redWindSpeed: 45,
          },
        },
      ],
      maps:maps.maps
}
},
mounted() {
  this.fetchSomething()
},
methods:{
  async fetchSomething() {
    let url = 'https://dcc5.modext.ru:8088/dataserver/api/v2/dataseries/list?flag=lastdata'
/*--header 'X-Ticket: ST-test' \
--header 'Content-Type: application/json' \
--data '{
  "sid": ["RU80-980401-0000", "RU98-998401-0000"],
  "dst": ["FRC-WTH-DATA-XXXXXXXXXXX"]
}'
*/
    const res = await this.$axios.$post(url, {
    headers: {
      'X-Ticket': 'ST-test',
      'Content-Type': 'application/json'
    },
    data: {
  "sid": ["RU80-980401-0000", "RU98-998401-0000"],
  "dst": ["FRC-WTH-DATA-XXXXXXXXXXX"]
}
})
    console.log('res', res)
  }
}
};
</script>

<style>
.map {
  width: 700px;
  height: 150px;
  padding: 5% 40%;
}
</style>
