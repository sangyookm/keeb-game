<template>
  <router-view/>
</template>

<script>
import 'normalize.css'
export default {
}
</script>

<style lang="scss">
html, body {
  background-color: #0a0a0a;
  width: 100%;
  height: 100%;
}

*, *:after, *:before {
  box-sizing: border-box;
}

#app {
  width: 100%;
  height: 100%;
  // color: #2c3e50;
  display: flex;
}
</style>
