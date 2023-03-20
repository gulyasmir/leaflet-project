<template>
  <div class="map">
    <LeafletMapComponent :citiesList="cities" :mapsList="maps" />
  </div>
</template>

<script>
export default {
  name: "IndexPage",
  data() {
    return {
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
      maps: [
        {
          id: 1,
          mapURL: "mapADG.svg",
          bounds: [
            [45.213278, 37.772987],
            [43.949287, 41.32026],
          ],
          title: "Адыгея",
          center:[44.00,39.00],
        },
        {
          id: 2,
          mapURL: "mapADG.svg",
          bounds: [
            [45.213278, 37.772987],
            [43.949287, 41.32026],
          ],
          title: "Адыгея",
          center:[44.00,39.00],
        },
      ],
    };
  },
};
</script>

<style>
.map {
  width: 700px;
  height: 150px;
  padding: 5% 40%;
}
</style>
