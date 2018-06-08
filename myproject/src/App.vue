<template>
  <div id="app">
    <weather></weather>
  </div>
</template>

<script>
  import weather from './components/weather'
export default {
  name: 'App',
  components:{
    weather
  }
}
</script>

<style>
  *{
    margin: 0;
    padding: 0;
  }
  /*body,html{*/
    /*width: 100%;*/
    /*height: 100%;*/
    /*!*font-size: 62.5%;*!*/
  /*}*/
  /*#app{*/
    /*width: 100%;*/
    /*height: 100%;*/
  /*}*/
</style>
