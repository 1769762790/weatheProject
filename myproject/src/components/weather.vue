<template>
  <div id="weather" ref="weather" v-cloak>
    <div class="today" ref="today" :class="classWind">
      <div class="currentCity">{{currentWeather.city}}</div>
      <div class="currentDate">今日 {{currentWeather.reporttime}} （实时：{{currentWeather.temperature}}℃）</div>
      <img v-if="currentWeather.dayImg" :src="['../../static/img/'+currentWeather.dayImg]" class="weather-icon">
      <div class="todayWeather" v-if="currentWeather.daytemp">{{currentWeather.nighttemp}}~{{currentWeather.daytemp}}℃</div>
      <div class="weather">{{currentWeather.weather}} {{currentWeather.winddirection}}风</div>
      <div class="pm">风力：{{currentWeather.windpower}}级</div>
    </div>
    <div class="futureWeather" ref="future">
        <ul class="weatherList">
          <li class="itemWeather" v-for="(item,index) in fCasts" @click.prevent="change(index)" :class="{'active':currentIndex===index}" ref="evDay">
            <div class="mainInfo">
              <div class="weekone">{{item.week}}</div>
              <div class="weatherPic"><img :src="['../../static/img/'+item.img]"></div>
              <div class="temperature">{{item.nighttemp}}/{{item.daytemp}}℃</div>
              <div class="weatherType">{{item.dayweather}}</div>
              <div class="cloud">{{item.daywind}}风</div>
            </div>
            <div class="hide" v-show="isShow && currentIndex==index">
              <div class="day">
                <div class="dayTitle">白天</div>
                <div class="dayWeatherDetil">
                  <div class="dayweather">{{item.dayweather}}</div>
                  <div class="daytemp">{{item.daytemp}}℃</div>
                  <div class="daywind">{{item.daywind}}风</div>
                  <div class="daypower">风力{{item.daypower}}级</div>
                </div>
              </div>
              <div class="daynight">
                <div class="nightTitle">夜晚</div>
                <div class="nightWeatherDetil">
                  <div class="nightweather">{{item.nightweather}}</div>
                  <div class="nightemp">{{item.nighttemp}}℃</div>
                  <div class="nightwind">{{item.nightwind}}风</div>
                  <div class="nightpower">风力{{item.nightpower}}级</div>
                </div>
              </div>
            </div>
          </li>
        </ul>
    </div>
    <div class="article">
      <p class="oneWord">{{this.oneWord}}</p>
      <p class="wordFrom">——{{this.wordFrom}}</p>
    </div>
  </div>
</template>

<script>
  import $ from 'jquery'
  import qs from 'qs'
  export default {
  name: 'weather',
  data () {
    return {
      currentWeather:[],//今天天气
      futureWeather:[],//未来总的天气
      currentIndex:'',//当前天气下标
      fCasts:[],//简化的未来天气
      isShow:false,//是否显示
      key:'0602f554ccff56553199cb55a948d811',//高徳key
      currentCityCode:'',//当前地区的adcode
      oneWord:'',//每日一句
      wordFrom:'',//每日一句出处
      location:'',//经纬度
      classWind:{//天气背景图
        forgbg:false,
        hazebg:false,
        snowbg:false,
        rainbg:false,
        cloudbg:false,
        clearbg:false,
        defaultbg:false,
        thunderbg:false,
        overcastbg:false,
        sanddustbg:false,
      }
    }
  },
  created () {
    //获取地址栏参数
    this.location = this.$route.query.location
    if(this.location){
      this.getCurrentCityLocation(this.location)
    }else {
      this.location = '125.32,43.92'
      this.getCurrentCityLocation(this.location)
    }

    this.getOneSestence()//每日一句
  },
  methods:{
    change(index){//点击给当前点击的天气添加样式并且控制显示与隐藏
      this.currentIndex = index
      this.isShow = true
      $(this.$refs.evDay[index]).animate({//通过jq的节点动画显示当前点击的li
        height: "17rem"
      }, 500);
      $(this.$refs.evDay[index]).siblings().animate({  //通过jq的节点动画隐藏当前点击的li的兄弟标签的高度
        height: "7rem"
      }, 500);

    },
    getCurrentCityLocation(locationAddress) {//获取城市经纬度，通过经纬度请求高徳api获得当前城市的adcode
      this.$http.get('http://restapi.amap.com/v3/geocode/regeo', {
        params: {
          location: locationAddress,
          key: this.key
        }
      })
        .then(res=>{
          this.currentCityCode = res.data.regeocode.addressComponent.adcode
          this.$http.all([this.getAllWeather(this.currentCityCode),this.getLiveWeather(this.currentCityCode)])
            .then(this.$http.spread((acct,perms)=>{
              // acct  未来天气数据
              console.log(acct)
              if (acct.data.status == 1){
                var data = acct.data
                this.futureWeather = data.forecasts[0]
                this.fCasts = this.futureWeather.casts
                this.fCasts.forEach((item,index,arr)=>{
                  arr[index].week=this.handleWeek(arr[index].week)
                  arr[index].img = this.weatherPic(arr[index].dayweather)
                })
              }
              if (perms.data.status == 1){
                //perms  当天天气数据
                console.log(perms)
                var data = perms.data
                this.currentWeather = data.lives[0]
                this.currentWeather.dayImg = this.weatherPic(this.currentWeather.weather)
                this.currentWeather.daytemp = this.futureWeather.casts[0].daytemp
                this.currentWeather.nighttemp = this.futureWeather.casts[0].nighttemp
                this.currentWeather.reporttime = this.reportTime(this.currentWeather.reporttime)
                this.currentWeather.weather = this.currentWeather.weather
                console.log(this.currentWeather)
                this.changeBg(this.currentWeather.weather)
              }
            }))
        })
    },
    /*
    * @params {String} key 高徳天气key
    * @params {String} city 高徳城市adcode
    * @params {String} extensions 显示未来的天气 all代表所有
    * */
    getAllWeather (cityCode) {//获取所有天气
      return this.$http.get('http://restapi.amap.com/v3/weather/weatherInfo',{
        params:{
          key:this.key,
          city:cityCode,
          extensions:'all'
        }
      })
    },
    getLiveWeather (cityCode) {//获取当前实时的天气
      return this.$http.get('http://restapi.amap.com/v3/weather/weatherInfo',{
        params:{
          key:this.key,
          city:cityCode,
        }
      })
    },
    getOneSestence(){//每日一句
      let params = {
        TransCode:'030111',
        OpenId:'123456789'
      }
      this.$http.post('https://api.hibai.cn/api/index/index',qs.stringify(params))
        .then(res=>{
          let data = res.data
          this.oneWord = data.Body.word
          this.wordFrom = data.Body.word_from
        })
        .catch(e=>{
          console.log(e)
        })
    },
    handleWeek(w){//星期转换
      let res = ''
      switch (w){
        case '7':
          res = '周日'
          break;
        case '6':
          res = '周六'
          break;
        case '5':
          res = '周五'
          break;
        case '4':
          res = '周四'
          break;
        case '3':
          res = '周三'
          break;
        case '2':
          res = '周二'
          break;
        default:
          res = '周一'
          break;
      }
      return res
    },
    weatherPic(wea){//天气图标
      var weatherIcon = ''
      switch (wea) {
        case '晴':
          weatherIcon = 'qing_0.png'
          break
        case '多云':
          weatherIcon = 'duoyun_0.png'
          break
        case '阴':
          weatherIcon = 'yin_0.png'
          break
        case '雷阵雨':
          weatherIcon = 'leizhenyu_0.png'
          break
        case '大雨':
          weatherIcon = 'dayu_0.png'
          break
        case '小雨':
          weatherIcon = 'xiaoyu_0.png'
          break
        case '中雨':
          weatherIcon = 'xiaoyu_0.png'
          break
        case '暴雨':
          weatherIcon = 'baoyu_0.png'
          break
        case '大雪':
          weatherIcon = 'daxue_0.png'
          break
        case '雨夹雪':
          weatherIcon = 'daxue_0.png'
          break
      }
      return weatherIcon;
    },
    reportTime(time){//转换日期格式
      var timearr = time.split(' ').slice(0,1).join('').replace(/-0/g,"-").split('-')
      var month = timearr.slice(1,2)
      var day = timearr.slice(2,3)
      var currentMonth = month.join('')
      var currentDay = day.join('')
      return currentMonth+'月'+currentDay+'日'
    },
    changeBg(weather){//改变背景
      for (let i in this.classWind) {
        this.classWind[i] = false
      }
      if (weather == '晴'){
        this.classWind.clearbg = true
      }else if (weather == '霾'){
        this.classWind.hazebg = true
      }else if (weather == '多云'){
        this.classWind.cloudbg = true
      }else if (weather == '阴天'){
        this.classWind.overcastbg = true
      }else if (weather == '轻雾'){
        this.classWind.forgbg = true
      }else if (weather == '阵雪'||'小雪'||'中雪'||'大雪'||'暴雪'){
        this.classWind.snowbg = true
      }else if (weather == '浮尘'||'扬沙'||'强沙尘暴'||'沙尘暴'){
        this.classWind.sanddustbg = true
      }else if (weather == '阵雨'||'小雨'||'中雨'||'大雨'||'暴雨'||'大暴雨'||'特大暴雨'){
        this.classWind.rainbg = true
      }else{
        this.classWind.defaultbg = true
      }
      // console.log(this.classWind)
    }
    }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import "weather.css";
</style>

