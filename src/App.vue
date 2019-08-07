<template>
  <div id="app">
    <main-container></main-container>
  </div>
</template>

<script>
  import Container from './components/Container.vue';
  export default {
    name: 'app',
    data () {
      return {
      }
    },
    components: {
      mainContainer : Container
    }
  }
</script>

<style>

</style>
