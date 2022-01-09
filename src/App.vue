<template>
  <div id="app">
    <ImportExport></ImportExport>
  </div>
</template>

<script>
import ImportExport from "./components/ImportExport"

export default {
  name: 'App',
  components: {ImportExport}
}
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
