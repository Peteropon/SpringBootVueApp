<template>
  <div id="app">
    <Todos />
    <footer class="info">
      <p>Based on a project by Evan You</p>
      <p>Modified for this tutorial by Andrew Hughes</p>
    </footer>
  </div>
</template>

<script>
import Todos from './components/Todos.vue'

export default {
  name: 'app',
  components: {
    Todos,
  }
}
</script>

<style>
  [v-cloak] { display: none}
</style>
