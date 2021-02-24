<template>
  <div id="app">
    <dashboard/>
  </div>
</template>

<script>
import Dashboard from './components/Dashboard'

export default {
  name: 'App',
  components: {
    Dashboard
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  text-align: center;
}

.columns {
  padding: 0 2rem;
}

nav {
  padding: 2rem;
  margin-bottom: 1rem;
}
</style>
