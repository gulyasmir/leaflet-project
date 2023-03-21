<template>
  <div class="info-block">
    <div v-if="tooltip">
      <img
        :src="setIcon(city.forecasts.forecastsIcon)"
        class="info-block__icon tooltip "
        :style="
          this.viewData === 'wind'
            ? 'transform: rotate(' +
              city.wind.iconRotate +
              'deg); width:30px; height:30px'
            : ''
        "
      />
      <div class="tooltip">
        <div v-if="viewData === 'forecasts'" class="black-weight-text">
          {{ city.forecasts.temperature }} °
        </div>
        <div v-else-if="viewData === 'wind'">
          <template>
            <div class="black-weight-text">
            <p>
              {{ city.wind.blackWindSpeed }}
              <span v-if="city.wind.redWindSpeed" class="red-weight-text">
                | {{ city.wind.redWindSpeed }}
              </span>
            </p>
          </div>
          </template>
        </div>
      </div>
    </div>
    <div v-else>
      <div v-if="viewData === 'forecasts'" class="black-weight-text">
        {{ city.forecasts.temperature }} °
      </div>
      <div v-else-if="viewData === 'wind'">
        <template>
          <div class="black-weight-text">
            <p>
              {{ city.wind.blackWindSpeed }}
              <span v-if="city.wind.redWindSpeed" class="red-weight-text">
                | {{ city.wind.redWindSpeed }}
              </span>
            </p>
          </div>
        </template>
      </div>

      <img
        :src="setIcon(city.forecasts.forecastsIcon)"
        class="info-block__icon"
        :style="
          this.viewData === 'wind'
            ? 'transform: rotate(' +
              city.wind.iconRotate +
              'deg); width:30px; height:30px'
            : ''
        "
      />
    </div>
  </div>
</template>

<script>
export default {
  name: "InfoBlock",
  props: {
    city: {},
    viewData: {
      type: String,
      default: "forecasts",
    },
    tooltip: {
      type: String,
      default: "",
    },
  },
  methods: {
    setIcon(forecastsIcon) {
      switch (this.viewData) {
        case "forecasts":
          return "images/forecasts-icons/" + forecastsIcon;
        case "wind":
          return "images/wind.svg";
    
        default:
          return "images/forecasts-icons/" + forecastsIcon;
      }
    },
  },
};
</script>

<style>
.info-block {
    min-width: 100px;
    padding: 0;
    margin: 0;
}
/*  размер иконок о погоде */
.info-block__icon { 
  width: 20px;
  height: 20px;
}
.info-block__icon.tooltip { 
  width: 35px;
  height: 35px;
}

.black-weight-text {
  font-size: 12px;
  color: #000;
  font-weight: 500;
  margin: 0;
  padding: 0;
}
.tooltip {
  margin-left: 35px;
  margin-top: -10px;
}
.tooltip .black-weight-text {
  font-size: 18px;
}
.red-weight-text {
  font-size: 16px;
  color: #f00;
  font-weight: 700;
  margin: 0;
  padding: 0;
}
</style>
