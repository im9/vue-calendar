<template>
  <div id="app">
    <Calendar />
  </div>
</template>

<script>
import Calendar from './components/Calendar.vue'

export default {
  name: 'App',
  components: {
    Calendar
  }
}
</script>

<style scoped>
#app {
  width: 400px;
  margin: auto;
  font-size: 12px;
}
</style>