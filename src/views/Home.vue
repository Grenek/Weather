<template>
  <div>
    <a href="/">Погода</a>
    <SearchBar />
    
  </div>
</template>
<script>
import SearchBar from "@/components/SearchBar.vue";

export default {
  name: "home",
  components: {
    SearchBar
  }
};
</script>
