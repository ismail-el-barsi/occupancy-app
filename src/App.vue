<template>
  <div id="app">
    <BuildingOccupancy />
  </div>
</template>

<script>
import BuildingOccupancy from "./components/BuildingOccupancy.vue";

export default {
  name: "App",
  components: {
    BuildingOccupancy,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
