<template>
  <div>
    <multiselect
      v-model="temp"
      label="city"
      track-by="city"
      placeholder="Введите город"
      open-direction="bottom"
      :options="options"
      :multiple="false"
      :searchable="true"
      :loading="isLoading"
      :clear-on-select="false"
      :close-on-select="true"
      :options-limit="30"
      :limit="10"
      :limit-text="limitText"
      :max-height="600"
      :show-no-results="false"
      :hide-selected="false"
      :custom-label="customLabel"
      @search-change="getInfoAndGeocode"
      @select="getTimeZone"
      @sliding-start="onSlideStart"
      @sliding-end="onSlideEnd"
    >
      <span slot="noOptions"></span>
    </multiselect>
    <b-carousel
      id="carousel-1"
      v-model="slide"
      :interval="0"
      controls
      indicators
      background="#000000"
      style="text-shadow: 1px 1px 2px #333;"
    >
      <b-carousel-slide v-for="item in forecast" v-bind:key="item.id" img-blank>
        <div class="output">{{ item[0].weekday }}</div>
        <div class="weathericons d-flex justify-content-center">
          <div v-for="i in item" v-bind:key="i.id">
            <div class="time">{{ i.time }}</div>
            <WeatherIcons :lvl="i.weather" :temperature="i.temperature"></WeatherIcons>
            <div class="temp">{{ i.temp }}</div>
          </div>
        </div>
      </b-carousel-slide>
    </b-carousel>
  </div>
</template>

<script>
import WeatherIcons from "@/components/WeatherIcons.vue";
import Multiselect from "vue-multiselect";
import axios from "axios";
var moment = require("moment");

export default {
  name: "SearchBar",
  components: { Multiselect, WeatherIcons },
  data() {
    return {
      slide: 0,
      sliding: null,
      forecast: [],
      options: [],
      isLoading: false,
      timezoneOffset: 0,
      temp: []
    };
  },
  methods: {
    limitText(count) {
      return `and ${count} other places`;
    },
    customLabel({ city, description }) {
      return `${city} – ${description}`;
    },

    onSlideStart(slide) {
      console.log("123");

      // this.$router.push({
      //   path: "/",
      //   query: {
      //     kek: "1",
      //     date: moment(new Date()).format("D-MM-YYYY")
      // lat: this.forecast.lat,
      // lon: this.forecast.lon
      //   }
      // });
    },
    onSlideEnd(slide) {
      alert("111");
    },
    getInfoAndGeocode(query) {
      let self = this;
      self.options = [];
      this.isLoading = true;
      if (query.length >= 3) {
        axios
          .get(`https://geocode-maps.yandex.ru/1.x/`, {
            params: {
              format: "json",
              apikey: "6e44cf80-f0d5-4c6c-801b-1c11ebb8fa6f",
              geocode: query
            }
          })
          .then(response => {
            response.data.response.GeoObjectCollection.featureMember.forEach(
              item => {
                self.options.push({
                  description: item.GeoObject.description,
                  city: item.GeoObject.name,
                  lat: item.GeoObject.Point.pos.split(" ")[0],
                  lon: item.GeoObject.Point.pos.split(" ")[1]
                });
              }
            );
            this.isLoading = false;
          });
      }
    },
    getWeatherForCity(city) {
      this.forecast = [];
      let self = this;
      let day = [];
      let temp = [];
      axios
        .get(`https://api.openweathermap.org/data/2.5/forecast`, {
          params: {
            lat: city.lon,
            lon: city.lat,
            APPID: "828beab3d73557dad05ad548fcded3ac"
          }
        })
        .then(response => {
          response.data.list.forEach(item => {
            let localizedTime = moment(item.dt_txt).utcOffset(
              this.timezoneOffset
            );
            if (temp !== this.getWeekday(localizedTime)) {
              day = [];
              day.push({
                weekday: this.getWeekday(localizedTime),
                time: this.getTime(localizedTime),
                weather: item.weather[0].main,
                temp: Math.round(item.main.temp - 273.15) + "°С"
              });
              self.forecast.push(day);
              temp = this.getWeekday(localizedTime);
            } else {
              day.push({
                weekday: this.getWeekday(localizedTime),
                time: this.getTime(localizedTime),
                weather: item.weather[0].main,
                temp: Math.round(item.main.temp - 273.15) + "°С"
              });
            }
          });
          this.isLoading = false;
          this.$router
            .replace({
              path: "/",
              query: {
                date: moment(new Date()).format("D-MM-YYYY"),
                lat: city.lat,
                lon: city.lon
              }
            })
            .catch(err => {});
        });
    },

    getTimeZone(query) {
      axios
        .get(`http://api.timezonedb.com/v2.1/get-time-zone`, {
          params: {
            key: "C1ALDCE8BIC4",
            format: "json",
            by: "position",
            lat: query.lon,
            lng: query.lat
          }
        })
        .then(response => {
          this.timezoneOffset = response.data.gmtOffset / 3600 + 3;
          this.getWeatherForCity(query);
        });
    },

    getWeekday(date) {
      let actDate = moment(date);
      // console.log(actDate);
      let dayOfWeek = actDate.day();
      return isNaN(dayOfWeek)
        ? null
        : [
            "Воскресенье",
            "Понедельник",
            "Вторник",
            "Среда",
            "Четверг",
            "Пятница",
            "Суббота"
          ][dayOfWeek];
    },

    getTime(date) {
      let actDate = moment(date);
      return `${actDate.hours()}:${actDate.minutes()}0`;
    }
  }
};
</script>
