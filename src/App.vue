<template>
  <div id="app">
    <Questions/>
  </div>
</template>

<script>
import Questions from '@/components/Questions.vue'

export default {
  name: 'app',
  components: {
    Questions
  }
}
</script>

<style>
body {
  background-color: hsl(0, 0%, 98%);
}

footer {
  background-color: hsl(211, 100%, 75%);
  margin-top: 90px;
  height: 100px;
}

figure {
  margin: 0;
}

:focus {
  outline: none;
}

* {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
