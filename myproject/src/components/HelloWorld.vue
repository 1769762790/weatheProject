<template>
  <div id="weather" ref="weather" v-cloak>
    <div class="today" ref="today">
      <div class="currentCity">{{currentWeather.city}}</div>
      <div class="currentDate">今日 {{currentWeather.reporttime}} （实时：{{currentWeather.temperature}}℃）</div>
      <img v-if="currentWeather.dayImg" :src="['../../static/img/'+currentWeather.dayImg]" class="weather-icon">
      <div class="todayWeather" v-if="currentWeather.daytemp">{{currentWeather.nighttemp}}~{{currentWeather.daytemp}}℃</div>
      <div class="weather">{{currentWeather.weather}} {{currentWeather.winddirection}}风</div>
      <div class="pm">风力：{{currentWeather.daywindpower}}</div>
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
  </div>
</template>

<script>
  import $ from 'jquery'
export default {
  name: 'weather',
  data () {
    return {
      currentWeather:[],//今天天气
      futureWeather:[],//未来总的天气
      currentIndex:'',//当前天气下标
      fCasts:[],//简化的未来天气
      isShow:false,//是否显示
    }
  },
  created () {
    //获取所有天气
    //this.getAllWeather()
    //获取当前天气
    //this.getLiveWeather()
    this.$http.all([this.getAllWeather(),this.getLiveWeather()])
      .then(this.$http.spread((acct,perms)=>{
        console.log(perms)
      if (acct.data.status == 1){
        var data = acct.data
        this.futureWeather = data.forecasts[0]
        this.fCasts = this.futureWeather.casts
        this.fCasts.forEach((item,index,arr)=>{
          arr[index].week=this.handleWeek(arr[index].week)
          arr[index].img = this.weatherPic(arr[index].dayweather)
        })
        console.log(this.fCasts)
      }
      if (perms.data.status == 1){
        var data = perms.data
        console.log(data)
        this.currentWeather = data.lives[0]
        this.currentWeather.dayImg = this.weatherPic(this.currentWeather.weather)
        console.log(this.currentWeather.windpower)
        this.currentWeather.daywindpower = this.WindPower(this.currentWeather.windpower)
        this.currentWeather.daytemp = this.futureWeather.casts[0].daytemp
        this.currentWeather.nighttemp = this.futureWeather.casts[0].nighttemp
        this.currentWeather.reporttime = this.reportTime(this.currentWeather.reporttime)
        console.log(this.currentWeather)
      }
    }))
  },
  mounted() {
    //计算底部高度
    // this.initHeight()
  },
  methods:{
    //计算底部未来天气的总高度，铺满剩余空间
    initHeight () {
      let futureHeight = this.$refs.weather.offsetHeight-this.$refs.today.offsetHeight+'px';
      this.$refs.future.style.height = futureHeight;
    },
    //点击给当前的天气添加样式并且控制显示与隐藏
    change(index){
      this.currentIndex = index
      this.isShow = true
      //通过jq的节点动画显示当前点击的li
      $(this.$refs.evDay[index]).animate({
        height: "31.97183vh"
      }, 500);
      //通过jq的节点动画隐藏当前点击的li的兄弟标签的高度
      $(this.$refs.evDay[index]).siblings().animate({
        height: "12.32394vh"
      }, 500);
    },
    /*
    * @params {String} key 高徳天气key
    * @params {String} city 高徳城市adcode
    * @params {String} extensions 显示未来的天气
    * */
     getAllWeather () {
      return this.$http.get('http://restapi.amap.com/v3/weather/weatherInfo',{
        params:{
          key:'0602f554ccff56553199cb55a948d811',
          city:'310101',
          extensions:'all'
        }
      })
      // .then((res)=>{
      //   if (res.data.status == 1) {
      //     let data = res.data
      //       this.futureWeather = data.forecasts[0]
      //       console.log(this.futureWeather)
      //       this.fCasts = this.futureWeather.casts
      //       this.fCasts.forEach((item,index,arr)=>{
      //         arr[index].week=this.handleWeek(arr[index].week)
      //         arr[index].img = this.weatherPic(arr[index].dayweather)
      //         //let w = arr[index].week;
      //         //let futureImg = this.weatherPic(arr[index].dayweather)
      //         //let Editweek = this.handleWeek(w)
      //         /*arr[index] = Object.assign({},arr[index],{
      //           // week: Editweek,
      //           img: futureImg
      //         })*/
      //       })
      //
      //
      //     console.log(this.fCasts)
      //   }
      // })
      // .catch((e)=>{
      //   console.log(e)
      // })
    },
    //获取当前实时的天气
    getLiveWeather () {
      return this.$http.get('http://restapi.amap.com/v3/weather/weatherInfo',{
        params:{
          key:'0602f554ccff56553199cb55a948d811',
          city:'310101',
        }
      })
      // .then((res)=>{
      //   console.log(res)
      //   let data = res.data
      //   if (data.status == '1') {
      //       this.currentWeather = data.lives[0]
      //
      //       this.currentWeather.dayImg = this.weatherPic(this.currentWeather.weather)
      //       this.currentWeather.daywindpower = this.WindPower(this.currentWeather.windpower)
      //       this.currentWeather.daytemp = this.futureWeather.casts[0].daytemp
      //       this.currentWeather.nighttemp = this.futureWeather.casts[0].nighttemp
      //     //let todayImg = this.weatherPic(this.currentWeather.weather)
      //     //let todayWindPower = this.WindPower(this.currentWeather.windpower)
      //       /*this.currentWeather = Object.assign({},this.currentWeather,{
      //         daytemp:this.futureWeather.casts[0].daytemp,
      //         nighttemp:this.futureWeather.casts[0].nighttemp,
      //         dayImg:todayImg,
      //         daywindpower:todayWindPower
      //       })*/
      //
      //     console.log(this.currentWeather)
      //   }
      // })
      // .catch((e)=>{
      //   console.log(e)
      // })
    },
    /*处理week*/
    handleWeek(w){
      let res = ''
      if(w == 5){
        res = '周五'
      }else if(w==6){
        res = '周六'
      }else if(w==7){
        res = '周日'
      }else if(w==1){
        res = '周一'
      }else if(w==2){
        res = '周二'
      }else if(w==3){
        res = '周三'
      }else if(w==4){
        res = '周四'
      }
      return res
    },
    /*风力转换*/
    WindPower(w) {
      var res = "";
      if(w == 0) {
        res = "1~2级";
      } else if(w == 1) {
        res = "3~4级";
      } else if(w == 2) {
        res = "4~5级";
      } else if(w == 3) {
        res = "5~6级";
      } else if(w == 4) {
        res = "6~7级";
      } else if(w == 5) {
        res = "7~8级";
      } else if(w == 6) {
        res = "8~9级";
      } else if(w == 7) {
        res = "9~10级";
      } else if(w == 8) {
        res = "10~11级";
      } else if(w == 9) {
        res = "11~12级";
      }
      return res;
    },
    /*天气图标*/
    weatherPic(weather){
        var weatherIcon = ''
        if (weather == '晴') {
          weatherIcon = 'qing_0.png'
        } else if (weather == '多云'){
          weatherIcon = 'duoyun_0.png'
        } else if (weather == '阴'){
          weatherIcon = 'yin_0.png'
        } else if (weather == '雷阵雨'){
          weatherIcon = 'leizhenyu_0.png'
        } else if (weather == '大雨'){
          weatherIcon = 'dayu_0.png'
        } else if (weather == '小雪'){
          weatherIcon = 'xiaoxue_0.png'
        } else if (weather == '阵雨'){
          weatherIcon = 'zhenyu_0.png'
        } else if (weather == '小雨'){
          weatherIcon = 'xiaoyu_0.png'
        } else if (weather == '暴雨'){
          weatherIcon = 'baoyu_0.png'
        } else if (weather == '大雪'){
          weatherIcon = 'daxue_0.png'
        } else if (weather == '雨夹雪'){
          weatherIcon = 'yujiaxue_0.png'
        }
        return weatherIcon;
      },
    /*转换日期格式*/
    reportTime(time){
        var timearr = time.split(' ').slice(0,1).join('').replace(/-0/g,"-").split('-')
        // var timelive = time.split(' ')
        // var nyr = timelive.slice(0,1)
        // var strnyr = nyr.join('')
        // var m = strnyr.replace(/-0/g,"-")
        // var timearr = m.split('-')
        var month = timearr.slice(1,2)
        var day = timearr.slice(2,3)
        var currentMonth = month.join('')
        var currentDay = day.join('')
        return currentMonth+'月'+currentDay+'日'
      }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  [v-cloak] {
    display: none;
  }
  #weather{
    width: 100%;
    height: 100%;
    background: linear-gradient(to bottom, #3e7dbe , #69aee5);
  }
  .today{
    display: flex;
    flex-direction: column;
    flex: 1;
    height: 40vh;
    justify-items: center;
    align-items: center;
    color: #eee;
  }
  .currentCity{
    margin:10px 0;
    font-size: 6vw;
  }
  .currentDate{
    font-size: 4.5vw;
  }
  .weather-icon{
    width: 25vw;
    height: 15vh;
  }
  .weatherPic img{
    width: 18.5633vw;
    height: 10vh;
  }
  .todayWeather{
    font-size: 5vw;
    margin-bottom:1vw;
  }
  .today .weather{
    font-size: 4.5vw;
    margin: 1vw 0;
  }
  .pm{
    font-size: 4.5vw;
    margin: 1vw 0;
  }
  .futureWeather{
    flex: 1;
    /*height: 100%;*/
    background: #64a7db;
  }
  .weatherList{
    list-style: none;
    list-style-type: none;
    width: 100%;
    height: 100%;
  }
  .weatherList .itemWeather{
    width: 100%;
    /*height: 12.3239vh;*/
    /*background: #69aee5;*/
    display: flex;
    flex-direction: column;
  }
  .itemWeather .mainInfo{
    display: flex;
    height: 12.32394vh;
    /*padding: 5px 0;*/
    /*box-sizing: border-box;*/
  }
  .weekone,.weatherPic,.temperature,.weatherType,.cloud{
    display: flex;
    flex: 1;
    align-items: center;
    justify-content: center;
    font-size: 4vw;
  }
  .itemWeather{
    color: #eee;
  }
  .active{
    background: #579bd7;
  }
  .hide{
    display: flex;
    width: 100%;
    height: 25.688155vh;
    background: rgba(117,180,219,.8);
  }
  .day,.daynight{
    flex: 1;
    /*flex-direction: row;*/
    flex-direction: column;
    justify-content: center;
    align-items: center;
    font-size: 4vw;
  }
  .dayWeatherDetil,.nightWeatherDetil{
    display: flex;
    flex: 1;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  .dayWeatherDetil{
    border-right: 1px solid cornflowerblue;
  }
  .dayTitle,.nightTitle{
    font-size: 4.5vw;
    text-align: center;
    margin: 1vh 0;
  }
</style>
