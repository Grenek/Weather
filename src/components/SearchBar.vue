<template>
  <div>
    <multiselect
      label="city"
      track-by="city + ', ' + description"
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
      :interval="4000"
      controls
      indicators
      background="#ababab"
      img-width="1024"
      img-height="480"
      style="text-shadow: 1px 1px 2px #333;"
      @sliding-start="onSlideStart"
      @sliding-end="onSlideEnd"
    >
      <!-- Text slides with image -->
      <b-carousel-slide
        caption="First slide"
        text="Nulla vitae elit libero, a pharetra augue mollis interdum."
        img-src="https://picsum.photos/1024/480/?image=52"
      ></b-carousel-slide>

      <!-- Slides with custom text -->
      <b-carousel-slide img-src="https://picsum.photos/1024/480/?image=54">
        <h1>Hello world!</h1>
      </b-carousel-slide>

      <!-- Slides with image only -->
      <b-carousel-slide img-src="https://picsum.photos/1024/480/?image=58"></b-carousel-slide>

      <!-- Slides with img slot -->
      <!-- Note the classes .d-block and .img-fluid to prevent browser default image alignment -->
      <b-carousel-slide>
        <img
          slot="img"
          class="d-block img-fluid w-100"
          width="1024"
          height="480"
          src="https://picsum.photos/1024/480/?image=55"
          alt="image slot"
        />
      </b-carousel-slide>

      <!-- Slide with blank fluid image to maintain slide aspect ratio -->
      <b-carousel-slide caption="Blank Image" img-blank img-alt="Blank image">
        <p>
          Lorem ipsum dolor sit amet, consectetur adipiscing elit. Suspendisse eros felis, tincidunt
          a tincidunt eget, convallis vel est. Ut pellentesque ut lacus vel interdum.
        </p>
      </b-carousel-slide>
    </b-carousel>

    <p class="mt-4">
      Slide #: {{ slide }}
      <br />
      Sliding: {{ sliding }}
    </p>
  </div>
</template>

<script>
import Multiselect from "vue-multiselect";
import axios from "axios";

export default {
  name: "SearchBar",
  components: { Multiselect },
  data() {
    return {
      slide: 0,
      sliding: null,
      weather: [],
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
      this.weather = [];
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
            self.weather.push({
              weather: item.weather[0].main,
              temp: item.main.temp,
              dateAndTime: item.dt_txt
            });
          });
          this.isLoading = false;
        });
    },
    onSlideStart(slide) {
      this.sliding = true;
    },
    onSlideEnd(slide) {
      this.sliding = false;
    }
  }
};
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
