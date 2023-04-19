<template>
  <div class="map-component">
    <div v-if="reload" class="full">
      <div class="loader-ring">
        <div class="loader-ring-light"></div>
        <div class="loader-ring-track"></div>
      </div>
    </div>
    <div v-else class="full">
      <div class="control-buttons">
        <button id="today" class="day-button active" @click="setDay('today')">
          Сегодня
        </button>
        <button id="tomorrow" class="day-button" @click="setDay('tomorrow')">
          Завтра
        </button>
        <button id="week" class="day-button" @click="setDay('week')">
          На неделю
        </button>
      </div>
      <div id="map-wrap" style="weight: 800px; height: 700px">
        <client-only>
          <LMap :zoom="selectedMap.zoom" :center="selectedMap.center" @click="getClickCoords($event)" :options="{
            doubleClickZoom: false,
            zoomControl: false,
            touchZoom: false,
            scrollWheelZoom: false,
            dragging: false,
            zoomSnap:0.1,
            zoomDelta:0.1
          }">
            <LTileLayer :url="url" :attribution="attribution" :bounds="selectedMap.bounds" :opacity="0" />
            <LImageOverlay :url="'/images/svg/' + selectedMap.mapURL + '.svg'" :bounds="selectedMap.bounds"
              crs="L.CRS.EPSG4326" :opacity="1" />
            <LControl position="topleft">
              <LeftControlList v-show="this.dayInfo !== 'week'"
                @selectControlButtons="onSelectControlButtons" />
              <LeftControlWeekList v-show="this.dayInfo === 'week'"
                @selectControlWeekButtons="onSelectControlWeekButtons" />
            </LControl>
            <LControl>
              <div class="map-buttons">
                <button :class="'forecasts-button ' + activeForecastsButton" @click="setViewData('forecasts')">
                  Прогноз
                </button>
                <button :class="'wind-button ' + activeWindButton" @click="setViewData('wind')">
                  Ветер
                </button>
              </div>
            </LControl>
            <div v-if="cities">
              <div v-for="city in cities" :key="city.id">
                <div>
                  <LMarker :lat-lng="[city.coords.latitude, city.coords.longitude]" :options="{ interactive: true }"
                    @click="getClickCoords($event)">
                    <LIcon :icon-size="dynamicSize" :icon-anchor="dynamicAnchor">
                      <InfoBlock :viewData="viewData" :city="timeInfo === 'day' ? city.info[dayIndex].day : city.info[dayIndex].night" />
                    </LIcon>
                    <LTooltip :options="{ direction: 'bottom', interactive: true }" @click="getClickCoords($event)">
                      <template>
                        <div class="tooltip-bg">
                          <div class="title">{{ city.title }}</div>
                          <div class="info">
                            <template>
                              <InfoBlock :viewData="viewData" :city="timeInfo === 'day' ? city.info[dayIndex].day : city.info[dayIndex].night" :tooltip="'tooltip'" />
                            </template>
                            <p class="min-max-info">
                              Min:
                              <span class="blue-text">{{ city.info[dayIndex].day.forecasts.min }}°
                              </span>
                              Max:
                              <span class="orange-text">{{ city.info[dayIndex].day.forecasts.max }}°
                              </span>
                            </p>
                          </div>
                        </div>
                      </template>
                    </LTooltip>
                  </LMarker>
                </div>
              </div>
            </div>
          </LMap>
        </client-only>
      </div>
      <div class="control-buttons">
        <select v-model="selectedMapId">
          <option value="0">--Выберите место--</option>
          <option v-for="map in maps" v-bind:value="map.id">
            {{ map.title }}
          </option>
        </select>
        <button v-if="selectedMapId > 1" @click="selectedMapId = 1">
          Вернуться на карту России
        </button>
      </div>
    
    </div>
  </div>
</template>

<script>
import {
  LMap,
  LTileLayer,
  LImageOverlay,
  LMarker,
  LIcon,
  LTooltip,
  LControl,
  LGeoJson,
} from "vue2-leaflet";
import { InfoControl, ReferenceChart, ChoroplethLayer } from "vue-choropleth";
import "leaflet/dist/leaflet.css";
import InfoBlock from "./InfoBlock";
import LeftControlList from "./LeftControlList";
import LeftControlWeekList from "./LeftControlWeekList";
import ChoroplethLayerComponent from "./ChoroplethLayerComponent";
import forecastStationsJson from "../static/json/forecast_stations.json";
import mapsJson from "../static/json/maps-full.json";
import * as axios from 'axios';
export default {
  name: "LeafletMapComponent",
  components: {
    LMap,
    LTileLayer,
    LImageOverlay,
    LMarker,
    LIcon,
    LTooltip,
    LControl,
    LGeoJson,
    ChoroplethLayerComponent,
    InfoControl,
    ReferenceChart,
    ChoroplethLayer,
    InfoBlock,
    LeftControlList,
    LeftControlWeekList,
  },

  data() {
    return {
      forecastStations: forecastStationsJson.items,
      cities: [],
      maps: mapsJson.maps,

      selectedMap: {
        id: 1,
        mapURL: "mapRUS",
        bounds: [
          [84.9468706507159, 15.154433208465578],
          [14.059857186546209, 195.04687070846558]
        ],
        title: "Россия",
        center: [66.58846086608366, 105.65947415779038
],
        zoom: 2.7,
      },
      reload: false,
      selectedMapId: 1,
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      staticAnchor: [16, 37],
      iconSize: 30,
      viewData: "forecasts",
      activeForecastsButton: "active",
      activeWindButton: "",
      dayInfo: "today",
      timeInfo: "day",
      dayIndex: 0,
      isDay:true
    };
  },
  mounted() {
    this.getStart();
  },
  methods: {
    setForecastsDayIcon(code) {
      let forecastsIcon = "";
      if (code == 1) forecastsIcon = "sun-sun.svg"; //1. ясно
      else if (code <= 3) forecastsIcon = "sun.svg"; //2. переменная без осадков
      else if (code == 4)
        forecastsIcon = "sun-cloud.svg"; //3. облачно без осадков
      else if (code > 4 && code < 25)
        forecastsIcon = "cloud.svg"; //4. переменная, снег
      else if (code > 24 && code < 41)
        forecastsIcon = "sun-rain.svg"; //5. облачно, снег
      else if ((code > 40 && code < 55) || (code > 61 && code < 76))
        forecastsIcon = "rain.svg"; //6. переменная, дождь
      else if ((code > 40 && code < 55) || (code > 61 && code < 76))
        forecastsIcon = "rain-rain.svg"; //6. переменная, дождь
      else if ((code > 54 && code < 62) || code > 75)
        forecastsIcon = "storm.svg"; //7. облачно, дождь
      return forecastsIcon;
    },
    setForecastsNightIcon(code) {
      let forecastsIcon = "";
      if (code == 1) forecastsIcon = "moon-moon.svg"; //1. ясно
      else if (code <= 3)
        forecastsIcon = "moon.svg"; //2. переменная без осадков
      else if (code == 4)
        forecastsIcon = "moon-cloud.svg"; //3. облачно без осадков
      else if (code > 4 && code < 25)
        forecastsIcon = "cloud.svg"; //4. переменная, снег
      else if (code > 24 && code < 41)
        forecastsIcon = "moon-rain.svg"; //5. облачно, снег
      else if ((code > 40 && code < 55) || (code > 61 && code < 76))
        forecastsIcon = "rain.svg"; //6. переменная, дождь
      else if ((code > 40 && code < 55) || (code > 61 && code < 76))
        forecastsIcon = "rain-rain.svg"; //6. переменная, дождь
      else if ((code > 54 && code < 62) || code > 75)
        forecastsIcon = "storm.svg"; //7. облачно, дождь
      return forecastsIcon;
    },
    createItemDayInfo(itemInfo) {
      let itemDayInfo = {
        frcDate: itemInfo.frcDate,
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
            redWindSpeed: null, // информации о порывах ветра нет
          },
        },
        night: {
          forecasts: {
            forecastsIcon: this.setForecastsNightIcon(itemInfo.ngtWCode),
            temperature: (itemInfo.maxTemp + itemInfo.minTemp) / 2,
            min: itemInfo.minTemp,
            max: itemInfo.maxTemp,
          },
          wind: {
            iconRotate: itemInfo.ngtWD,
            blackWindSpeed: itemInfo.ngtWS,
            redWindSpeed: null, // информации о порывах ветра нет
          },
        },
      };
      return itemDayInfo;
    },
    async getSettingJson() {
      const axios = require("axios");
      return axios
        .get("https://server-leaflet.herokuapp.com/download/setting-"+ this.selectedMap.mapURL + ".json", {
          headers: {
        "Allow": "*",
        "Access-Control-Allow-Origin": "*",
        "Content-Type": "application/json; charset=utf-8",
    }
        })
        .then((res) => res.data);
    },
    async getStart() {
      let settingJson = await this.getSettingJson(this.selectedMap.mapURL);
      let sid = settingJson.sources.towns;
      let dst = settingJson.dataseries.forecasts;
      this.cities = await this.getDataseriesCity(sid, dst);
    },
    async getDataseriesCity(sid, dst) {
      const axios = require("axios");
      let data = JSON.stringify({
        sid: sid,
        dst: [dst],
      });

      let config = {
        method: "post",
        url: "https://dcc5.modext.ru:8088/dataserver/api/v2/dataseries/list?flag=lastdata",
        headers: {
          "X-Ticket": "ST-test",
          "Content-Type": "application/json",
          Authorization: "Basic Og==",
        },
        data: data,
      };
      return await axios
        .request(config)
        .then((response) => {
          return response.data.response.dataseries.items
        })
        .then((citiesList) => {
          let number = 0;
          let result = [];
          for (const itemCity of citiesList) {
            let fetchData = itemCity.lastData.cv
            if (fetchData) {
              let info = fetchData.frcDay?.map(
                (itemInfo) => this.createItemDayInfo(itemInfo) //массив данных о погоде в этом городе
              );
              let itemCity = {
                coords: {
                  latitude: fetchData.stLat,
                  longitude: fetchData.stLon,
                },
                nameStation: fetchData.stName,
                info: info,
              };
         
              result.push({
                id: number,
                title: itemCity.nameStation,
                coords: itemCity.coords,
                info: itemCity.info,
                sid: itemCity.sid
              })
              number++;
            }
          }
          return result
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getClickCoords(eventData) {
      console.log(eventData.latlng.lat, eventData.latlng.lng)
      let lat = eventData.latlng.lat;
      let lng = eventData.latlng.lng;
      let selectMap = this.maps.find(
        (item) =>
          item.id > 1 &&
          lat < item.bounds[0][0] &&
          lat > item.bounds[1][0] &&
          lng > item.bounds[0][1] &&
          lng < item.bounds[1][1]
      );
      if (selectMap !== undefined) {
        this.selectedMapId = selectMap.id
      }
    },
    setDay(day) {
      let dayButtons = document.getElementsByClassName("day-button");
      Array.from(dayButtons).forEach((element) => {
        element.classList.remove("active");
      });
      document.getElementById(day).classList.add("active");
      if (day !== "week") {
        let timeInfo = this.isDay ? 'day' : 'night'
        let data = {
          dayInfo: day,
          timeInfo: timeInfo,
        };
        this.onSelectControlButtons(data);
      } else {
        this.dayInfo = day;
      }
    },

    onSelectControlButtons(data) {
      this.dayInfo = data.dayInfo;
      this.timeInfo = data.timeInfo;
      this.dayIndex = this.dayInfo === "today" ? 0 : 1;
      this.isDay = this.timeInfo === 'day' ? true : false
    },
    onSelectControlWeekButtons(data) {
      this.dayIndex = data.dayIndex;
      this.timeInfo = data.timeInfo;
      this.isDay = this.timeInfo === 'day' ? true : false
    },
    setMap(mapId) {
      this.selectedMap = this.maps.find(function (elem) {
        return elem.id === mapId;
      });
    },
    setViewData(type) {
      this.viewData = type;
      this.activeForecastsButton = type === "forecasts" ? "active" : "";
      this.activeWindButton = type === "wind" ? "active" : "";
    },
    mapIconStyle(iconRotate) {
      return this.viewData === "wind"
        ? "transform: rotate(" + iconRotate + "deg); width:30px; height:30px"
        : "";
    },
    setNewMap() {
      this.setMap(this.selectedMapId);
      this.getStart();
      this.reload = false;
    },
  },
  watch: {
    selectedMapId() {
      this.reload = true;
      setTimeout(this.setNewMap, 500);
    },
  },
  computed: {
    dynamicSize() {
      return [this.iconSize, this.iconSize / 1.15];
    },
    dynamicAnchor() {
      return [this.iconSize / 2, this.iconSize / 1.15];
    },
  },
};
</script>

<style>
@keyframes rotate-360 {
  from {
    -webkit-transform: rotate(0);
    -ms-transform: rotate(0);
    -o-transform: rotate(0);
    transform: rotate(0);
  }

  to {
    -webkit-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    -o-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}

.loader-ring-light {
  top: 50%;
  left: 63%;
  position: absolute;
  margin: -100px 0 0 -100px;
  width: 200px;
  height: 200px;
  border-radius: 200px;
  -webkit-box-shadow: 0 5px 0 #1a7ac7 inset;
  box-shadow: 0 5px 0 #1a7ac7 inset;
  animation: rotate-360 2s linear infinite;
}

#atmogramme .loader-ring {
  opacity: 0;
  transition: 0.5s all;
}

.loader-ring {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #fff;
  z-index: 555;
  pointer-events: none;
}

.full {
  height: 100%;
  width: 100%;
}

.leaflet-control-attribution {
  opacity: 0;
}

.map-component {
  width: 100%;
  height: 100%;
}

.black-weight-text {
  font-size: 12px;
  color: #333333;
  font-weight: 700;
  margin: 0;
  padding: 0;
}

.tooltip-bg {
  opacity: 0.8;
  background-color: #fff;
  padding: 5px 20px;
  border-radius: 3px;
}

.title {
  font-size: 18px;
  font-weight: 600;
  color: #036ba1;
  padding: 5px;
}

.min-max-info {
  padding: 5px 0 0 0;
  margin: 0;
  font-size: 12px;
  font-weight: 500;
}

.blue-text {
  color: #0381a1;
}

.orange-text {
  color: #f47a20;
}

button {
  cursor: pointer;
}

.map-buttons button {
  width: 16px;
  height: 11px;
  padding: 5px 10px;
  border: none;
  width: 150px;
  height: 30px;
  color: #fff;
  font-weight: 600;
  font-size: 14px;
}

.map-buttons button.active {
  color: #036ba1;
}

.forecasts-button {
  background: #036ba1;
  background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAgCAYAAABgrToAAAAAAXNSR0IArs4c6QAAAERlWElmTU0AKgAAAAgAAYdpAAQAAAABAAAAGgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAKKADAAQAAAABAAAAIAAAAABKli1tAAAENElEQVRYCeWYWWxNURSGW2NjqFA1VEkH0hJN2oggEWmLBDFUiCESEUoknoQHoQ+GmBLhTeKFiAdEIiHxRvCAVFM1U0MUVTU3NbTV4fr+5u7b02OfU71OvVjJl73PWmuvvc6e7j43JuZ/k1AoNArm294bfTpsgO42+z/R0bmRXGeHKJV4TdiY4LT51bv5GaO0XaVdC0xztR/PczN8gJ4um+djVyR4gd5CMMvV62KeB0FJbGxstcsW/CNTdSk8XWOc0dHlQ6NsLr1U0u936RPRPYG7Tv1f1wn4EprhI0xyBuR5O2x06Zajuwm9jZ56X7gOrS9EmWRspow1lc6WBBtBm/OQDS9gIVN3jzIG2wCKGZAGOfADyqAUivHDJdSfegXItwamo79DGZwoEfip3pCJiky5BsrhC7SAkQYq0p2EZJgIkg+Q65VV1CNoAhJcU5YKz+Es5EIPeAca2WLIgLEwBDRi8s2HTGhm5C5TBickpTlKgSzIhng4BLWgET0IkbVmeka3FiRau5JkYwukJGAa7IJHoE1SCa/gXbj8TLnPrzPs46ECmuCo8aWeB5vhCkg0K38uNCgEJaAR0tp6D6+hCupBUg03YLhfZOwrQLH0cqNhJUiUtBndAr8Y7Ww02gv6mVJyevvdMBUyYDKsAi1+HTlG2p2PzoA49ALNwHdYDbNBUgw7YKbT37eOs37cFUw7sBSGejXANg807Rrhu+C5xrBdBEm7g9srtlVP4yR4Bho5TclvC9/dEB/nGjvCs/WkQC+b5LA7hvO5o9/iZTiPhEoYy3HQ4Gxsq+NzH30h1MIcGE4SIgWGgulTm0A/h1UQnRDsFmjR7ulMBPy1xp5CHfwArVttCC2V+1AE2mDaaHmwCCSnoADSO+wPpx5ggkzpsIHLgbanQS+nHf4GFOstNEFDGG2SODgG5iTQWhf6CbULRiU3DsxR0rkzibC0TQbtyk2QCQkwBrRLn4ASlegk6A7ajGdAx5akyJodhiVwG7Qb9YZfIcvqHKWSePHhuLrFaClETgbqg/UMZq229YJyJ+i80xRozWgN6W0D/4Ygpk4ITb0GYUtbFh41nJaCktNhewLSwHpEeITotJr4B0CjWOLbGId+8BA0Ypd8nQM00lcOaM1ptvrYQpt51o04EfRBs8Dm2EW6L8RtDjPQ1odJULdj3dOqOWi/2Ry7SDeMuHGgNf7J1odJsAljPVjfwtYwIN064gyGFwyM+rcL858K2lE6SEfbvYLV0k866CjTJVeJegsO+mm6BrqF3IHIueTdKnoL8aeE+6JovSHFe0WLHCM46rvhMWi6dSnYBmXwE4IQ/dugb5L1MAE0COo/k+ktp7RKJEFZSXIyxXFIAS1c7bJGCELUV1/Q5+ZneApzSe4jpae0S1BeJKk/draCPiP1Tatv1iBEVyvNzAM4BedIro7SV34B0Hlk8hI4MvAAAAAASUVORK5CYII=");
  background-repeat: no-repeat;
  background-position: left 10px center;
  background-size: 25px;
  border-radius: 4px 0 0 4px;
}

.forecasts-button.active {
  background: #fff;
  background-image: url("data:image/png;base64,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");
  background-repeat: no-repeat;
  background-position: left 10px center;
  background-size: 25px;
}

.wind-button {
  background: #036ba1;
  background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAYCAYAAACbU/80AAAAAXNSR0IArs4c6QAAAERlWElmTU0AKgAAAAgAAYdpAAQAAAABAAAAGgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAIKADAAQAAAABAAAAGAAAAAA915G0AAABv0lEQVRIDc2WzytEURTHvUmjZKVsJCUbFCkiCwsbG6Gs2SgLZWFhZ6lsTVn6B9QslDDNGrOwsyJrs2AzNBaimetzZu7U6Xn3WrgvTn3n/Lrz/Z477773pq0tZTPGZMAOuAGv4BKspyzbpEdIxAsgyY5SHwLVLaVcIy6rXMLFVIdA4N4KvuBHRQw/Bz5tvZjaAAh0Atm12KEWIj9tVI15yuhG4PgDvmfLee7grjvqYcrssg/MajZbe7O/QCHSTR2zYJC8S9d+GYvWMNgDwi220XSxT8RX7YRpuhLkkesMjMdmCp0WIFyOosi0O5hz1LtByEtg4HsA16Ao4vi/t4jrsMQYmyAbaBy5te5ACeTZac3LywCPKZ40OWgDvgHkEB4D/5Q+Bn9vhvYZQzh/3cZzgAUdLHQdSL/E965sagzIQZ607V0uxb6Nwzs2sCbQzOQ9QF5CYle6FzSGfKIh0fyY0uSULmyvqus6dj2I9JqfYn0/L8QW99q8EquHS9lhFrReLlVieYyPgAPQsnw4xQQmVLZbSgn+ndpQwtfClRCQ/345UAfaKiQrPiXn69j3JVcPsWl686Af3IITbr8y/v/aF8GOLX4xFE/HAAAAAElFTkSuQmCC");
  background-repeat: no-repeat;
  background-position: left 10px center;
  background-size: 20px;
  border-radius: 0 4px 4px 0;
  margin-left: -3px;
}

.wind-button.active {
  background: #fff;
  background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAYCAYAAACbU/80AAAAAXNSR0IArs4c6QAAAERlWElmTU0AKgAAAAgAAYdpAAQAAAABAAAAGgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAIKADAAQAAAABAAAAGAAAAAA915G0AAACrklEQVRIDc1WTWhTQRCe2ZfXRA30ILm0Iki1IoIGFEUUBA8iCO0thzZpTj1Ikyq99hKoeK35OYpikkJpDx60Vk8epBUvgj9gsAcvCmIDVqo2JtkdZ19MStr3UjR54kKyuzOz883uzH77EFxuiQSJ68X8BBCFCOEwErxCQXcq6ehtDY1u4lvgq7kFArq0DUfALZmOjroagBnPxZRSaQscQXH/CQh6GsEIHBCNiQsDUhSrucWvBhhBmYn2ouG5AIhVLUeimGsB9CTu7yakQ1YASLlyJvxaj6up4SdcB4t6TIRB1wLoB3+Zj/uzBkIhFnS/vZFytQZ2xfP7KkAHqunI0zq4lpVJFXj7exDwkWMA3vhsn4KKv76w7d4QiFIdkaSm+GT6tD8Bwv4WdMXyYUky1zZoCweI+KySjpy1rQEF8niLtW2rEGHRh75BDoI8dt5MNJJlkHu5fDqXAuKaB3wHBi5VkuHHGtwO+5/L0IjlBpgQrjApd3UIXfHW3hoEy4PnffPzoZBs5ReNsbsf2KC3ldHf6nShcY6HS5mR904+BOdllnPdMkqnxTvJOe1nmHMfHE3MOZ6uxQMH4w+9696ibUHuBLJVL0W3WPuxfgxA3eQATmo9M+Ekk9GNrbaWzk74pzJzPBvRayqpkQZ3+ONzgQ1VWmHG7+Zrt1TNRM/Z+bXlATtDJ5k5PnNCScrqnxmfOVW3+5YOrfJNW9ZzvoCOvNJ2ACB10dcagbxcH9d6/P3205dm+eas7bz3B7xvCsXSd/24kIIJpvEVvtIvFKhR/h6wds6F9nwTsnnk+Bg1m7WemWPZawpo2tYK4acBZrCcGSrY6dtPAXudDERSXGhJrvdGOiwwhDX++hlyAtc2HTkBC4z/zKvZ0yTpIpP+fn5qX3o83nsb06GPdf1/2f8CBEfns8DB3a8AAAAASUVORK5CYII=");
  background-repeat: no-repeat;
  background-position: left 10px center;
  background-size: 20px;
}

.control-buttons {
  width: 100%;
}

.control-buttons button {
  background: #fff;
  border-radius: 4px 0 0 4px;
  color: #036ba1;
  width: 30%;
  padding: 5px 10px;
  border: none;
  height: 30px;
  font-weight: 600;
  font-size: 14px;
}

.control-buttons button:hover {
  background: #fff;
  color: #db0084;
}

.control-buttons button.active {
  background: #db0084;
  color: #fff;
}
</style>
