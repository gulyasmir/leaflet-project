<template>
  <div class="map">
    <LeafletMapComponent :citiesList="cities"  :mapsList="maps" />
  </div>
</template>

<script>
import forecastStationsJson from '../static/json/forecast_stations.json'
import mapsJson from '../static/json/maps-full.json'
export default {
  name: "IndexPage",
  
  data() {
    return {
      forecastStations: forecastStationsJson.items,
      cities:[],
      maps: mapsJson.maps
    }
  },
  mounted() {
    this.getData(this.forecastStations)
  },
  methods: {
    setForecastsDayIcon(code) {
      let forecastsIcon = ''
      if (code == 1) forecastsIcon = 'sun-sun.svg'          //1. ясно
      else if (code <= 3) forecastsIcon = 'sun.svg'            //2. переменная без осадков
      else if (code == 4) forecastsIcon = 'sun-cloud.svg'            //3. облачно без осадков
      else if (code > 4 && code < 25) forecastsIcon = 'cloud.svg'  //4. переменная, снег
      else if (code > 24 && code < 41) forecastsIcon = 'sun-rain.svg' //5. облачно, снег
      else if ((code > 40 && code < 55) || (code > 61 && code < 76)) forecastsIcon = 'rain.svg' //6. переменная, дождь
      else if ((code > 40 && code < 55) || (code > 61 && code < 76)) forecastsIcon = 'rain-rain.svg' //6. переменная, дождь
      else if ((code > 54 && code < 62) || (code > 75)) forecastsIcon = 'storm.svg' //7. облачно, дождь
      return forecastsIcon
    },
    setForecastsNightIcon(code) {
      let forecastsIcon = ''
      if (code == 1) forecastsIcon = 'moon-moon.svg'          //1. ясно
      else if (code <= 3) forecastsIcon = 'moon.svg'            //2. переменная без осадков
      else if (code == 4) forecastsIcon = 'moon-cloud.svg'            //3. облачно без осадков
      else if (code > 4 && code < 25) forecastsIcon = 'cloud.svg'  //4. переменная, снег
      else if (code > 24 && code < 41) forecastsIcon = 'moon-rain.svg' //5. облачно, снег
      else if ((code > 40 && code < 55) || (code > 61 && code < 76)) forecastsIcon = 'rain.svg' //6. переменная, дождь
      else if ((code > 40 && code < 55) || (code > 61 && code < 76)) forecastsIcon = 'rain-rain.svg' //6. переменная, дождь
      else if ((code > 54 && code < 62) || (code > 75)) forecastsIcon = 'storm.svg' //7. облачно, дождь
      return forecastsIcon
    },
    createItemDayInfo(itemInfo) {
      let itemDayInfo = {
        frcDate:itemInfo.frcDate,
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
