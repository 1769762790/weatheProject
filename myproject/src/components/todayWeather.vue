<template>
  <div>111</div>
</template>

<script>
    export default {
        name: "todayWeather",
        created(){
          this.getWeatherLive();
        },
        methods:{
          getWeatherLive(){
            $.ajax({
              type: 'get',
              url: 'http://api.k780.com/?app=weather.today&weaid=1&appkey=34007&sign=737ee5e6fb78e5d0152977d18cb55709&format=json&jsoncallback=data',
              dataType: 'jsonp',
              jsonp: 'callback',
              jsonpCallback: 'data',
              success: (data)=>{
                console.log(data)
              },
              error: function() {
                alert('fail');
              }
            });
          }
        }
    }
</script>

<style scoped>

</style>
