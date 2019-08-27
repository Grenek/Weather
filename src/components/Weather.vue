<template>
  <div>
    <SearchBar />
  </div>
</template>

<script>
import SearchBar from "@/components/SearchBar.vue";
import axios from "axios";

export default {
  name: "weather",

  methods: {
    getWeatherForCity() {
      axios.get(`https://api.openweathermap.org/data/2.5/forecast`, {
          params: {
            lat: selectedOption.lat,
            lon: selectedOption.lon,
            APPID: "828beab3d73557dad05ad548fcded3ac"
          }
        })
    }
  }
}
</script>