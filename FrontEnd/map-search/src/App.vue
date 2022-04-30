<script setup>
import TypeaheadMapSearch from "./components/TypeaheadMapSearch.vue";
</script>

<template>
  <TypeaheadMapSearch
    class="usmap-search"
    placeholder="Please enter state to search..."
  ></TypeaheadMapSearch>
</template>

<style>
@import "./assets/base.css";

#app {
  padding: 2rem;

  font-weight: normal;
}

body {
  background-color: #202124;
}

strong {
  font-weight: bolder;
}

.usmap-search {
  height: 100%;
}

html,
body,
#app {
  height: 95%;
}
</style>
