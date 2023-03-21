<template>
  <div class="map">
    <LeafletMapComponent :citiesList="cities1"  :citiesList1="cities"  :mapsList="maps" />
  </div>
</template>

<script>
import forecastStationsJson from '../static/json/forecast_stations.json'
import mapsJson from '../static/json/maps.json'
export default {
  name: "IndexPage",
  
  data() {
    return {
      forecastStations: forecastStationsJson.items,
      cities:[],

      cities1: [
        {
          id: 1,
          title: "Москва",
          coords: {
            latitude: 55.7522,
            longitude: 37.6156,
          },
          info: [

          ],
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
      maps: mapsJson.maps
    }
  },
  mounted() {
    this.getData(this.forecastStations)
  },
  methods: {

    setForecastsDayIcon(code) {
      let forecastsIcon = ''
      if (code == 1) forecastsIcon = '1.svg'          //1. ясно
      else if (code <= 3) forecastsIcon = '2.svg'            //2. переменная без осадков
      else if (code == 4) forecastsIcon = '3.svg'            //3. облачно без осадков
      else if (code > 4 && code < 25) forecastsIcon = '4.svg'  //4. переменная, снег
      else if (code > 24 && code < 41) forecastsIcon = '5.svg' //5. облачно, снег
      else if ((code > 40 && code < 55) || (code > 61 && code < 76)) forecastsIcon = '6.svg' //6. переменная, дождь
      else if ((code > 54 && code < 62) || (code > 75)) forecastsIcon = '7.svg' //7. облачно, дождь

      return forecastsIcon
    },
    setForecastsNightIcon(code) {
      let forecastsIcon = ''
      if (code == 1) forecastsIcon = '10.svg'          //1. ясно
      else if (code <= 3) forecastsIcon = '20.svg'            //2. переменная без осадков
      else if (code == 4) forecastsIcon = '30.svg'            //3. облачно без осадков
      else if (code > 4 && code < 25) forecastsIcon = '40.svg'  //4. переменная, снег
      else if (code > 24 && code < 41) forecastsIcon = '50.svg' //5. облачно, снег
      else if ((code > 40 && code < 55) || (code > 61 && code < 76)) forecastsIcon = '60.svg' //6. переменная, дождь
      else if ((code > 54 && code < 62) || (code > 75)) forecastsIcon = '70.svg' //7. облачно, дождь

      return forecastsIcon
    },
    createItemDayInfo(itemInfo) {
      let itemDayInfo = {
        day: {
          forecasts: {
            forecastsIcon: this.setForecastsDayIcon(itemInfo.dayWCode),
            temperature: (itemInfo.maxTemp + itemInfo.minTemp) / 2,
            min: itemInfo.minTemp,
            max: itemInfo.maxTemp,
          },
          wind: {
            iconRotate: itemInfo.dayWD,
            blackWindSpeed: itemInfo.dayWS,
            redWindSpeed: null // информации о порывах ветра нет
          }
        },
        night: {
          forecasts: {
            forecastsIcon: this.setForecastsNightIcon(itemInfo.dayWCode),
            temperature: (itemInfo.maxTemp + itemInfo.minTemp) / 2,
            min: itemInfo.minTemp,
            max: itemInfo.maxTemp,
          },
          wind: {
            iconRotate: itemInfo.dayWD,
            blackWindSpeed: itemInfo.dayWS,
            redWindSpeed: null // информации о порывах ветра нет
          }
        }
      }
      return itemDayInfo
    },
    async getData(forecastStations) {
      const result = [];
      let idCity = 0
      for (const item of forecastStations) {
      

        const axios = require('axios');
        let data = JSON.stringify({
          "sid": [item.sid],
          "dst": [
            "FRC-WTH-DATA-XXXXXXXXXXX"
          ]
        });

        let config = {
          method: 'post',
          maxBodyLength: Infinity,
          url: 'https://dcc5.modext.ru:8088/dataserver/api/v2/dataseries/list?flag=lastdata',
          headers: {
            'X-Ticket': 'ST-test',
            'Content-Type': 'application/json',
            'Authorization': 'Basic Og=='
          },
          data: data
        };

        axios.request(config)
          .then((response) => {
            return response.data.response.dataseries.items[0].lastData.cv.frcDay
          }).then((fetchData) => {
            let info = fetchData?.map(itemInfo => this.createItemDayInfo(itemInfo))
            let itemCity = {
              id: idCity,
              title: item.name,
              coords: {
                latitude: item.loc.lat,
                longitude: item.loc.lon,
              },
              info: info
            }
            result.push(itemCity)
            idCity++
          })
          .catch((error) => {
            console.log(error);
          })
      }
      this.cities  = result
    }
  }
}
</script>

<style>
.map {
  width: 700px;
  height: 150px;
  padding: 5% 40%;
}
</style>
