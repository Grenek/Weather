<template>
  <div>
    <multiselect
      label="city"
      track-by="city"
      placeholder="Type to search"
      open-direction="bottom"
      :options="options"
      :multiple="false"
      :searchable="true"
      :loading="isLoading"
      :clear-on-select="true"
      :close-on-select="true"
      :options-limit="30"
      :limit="10"
      :limit-text="limitText"
      :max-height="600"
      :show-no-results="true"
      :hide-selected="false"
      :custom-label="customLabel"
      @search-change="getInfoAndGeocode"
      @select="getWeatherForCity"
    ></multiselect>
    <b-carousel
      id="carousel-1"
      v-model="slide"
      :interval="0"
      controls
      indicators
      background="#000000"
      img-width="1024"
      img-height="480"
      style="text-shadow: 1px 1px 2px #333;"
      @sliding-start="onSlideStart"
      @sliding-end="onSlideEnd"
    >
      <b-carousel-slide v-for="item in forecast" v-bind:key="item.id" img-blank>
        <div class="output">{{ item.dateAndTime }}</div>
          <WeatherIcons :lvl="item.weather" :temperature="item.temperature">
          </WeatherIcons>
      </b-carousel-slide>
    </b-carousel>
  </div>
</template>

<script>
import WeatherIcons from "@/components/WeatherIcons.vue";
import Multiselect from "vue-multiselect";
import axios from "axios";

export default {
  name: "SearchBar",
  components: { Multiselect, WeatherIcons },
  data() {
    return {
      slide: 0,
      sliding: null,
      forecast: [],
      options: [],
      isLoading: false
    };
  },
  methods: {
    limitText(count) {
      return `and ${count} other places`;
    },
    customLabel({ city, description }) {
      return `${city} – ${description}`;
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
      axios
        .get(`https://api.openweathermap.org/data/2.5/forecast`, {
          params: {
            lat: city.lat,
            lon: city.lon,
            APPID: "828beab3d73557dad05ad548fcded3ac"
          }
        })
        .then(response => {
          response.data.list.forEach(item => {
            self.forecast.push({
              weather: item.weather[0].main,
              temp: item.main.temp,
              dateAndTime: item.dt_txt
            });
          });
          this.isLoading = false;
        });
    },
    // onSlideStart(slide) {
    //   this.sliding = true;
    // },
    // onSlideEnd(slide) {
    //   this.sliding = false;
    // }
  }
};
</script>
