<template>
  <router-view/>
</template>

<script>
import 'normalize.css'
export default {
}
</script>

<style lang="scss">

:root {
  --game-font-size: 24px;
  --game-line-height: 1.35;
  --game-show-rows: 5;
  --game-word-gap-v: 2px;
  --game-word-gap-h: 5px;
  --game-bg-color: #0a0a0a;
}

html, body {
  background-color: var(--game-bg-color);
  width: 100%;
  height: 100%;
  overflow: hidden;
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
