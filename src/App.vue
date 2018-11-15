<template>
  <div id="app">
    <Game />
  </div>
</template>

<script>
import Game from './components/Game.vue'

export default {
  name: 'app',
  components: {
    Game
  }
}
</script>

<style lang="scss">
* {
  box-sizing: border-box;
  user-select: none;
}

#app {
  max-width: 100vw;
}
</style>
