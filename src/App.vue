<template>
  <div id="app">
    <awesome-cropper></awesome-cropper>
  </div>
</template>

<script>
import AwesomeCropper from './components/AwesomeCropper'
export default {
  name: 'app',
  components: {
    AwesomeCropper
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
  margin-top: 60px;
}
</style>
