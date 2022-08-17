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
  background-color: #1BAC81;
  width: 100%;
  height: 100%;
}
#app {
  width: 100%;
  height: 100%;
  color: #2c3e50;
  display: flex;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
