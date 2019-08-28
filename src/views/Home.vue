<template>
  <div>
    <h2>Погода</h2>
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
