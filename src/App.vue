<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <md-table>
      <md-table-row>
        <md-table-head>City</md-table-head>
        <md-table-head>Current AQI</md-table-head>
        <md-table-head>Last Updated</md-table-head>
      </md-table-row>
      <md-table-row v-for="(value,key) in tableData" :key="key" @click.stop="openChart(value.series)">
        <md-table-cell>{{key}}</md-table-cell>
        <md-table-cell :style="colorValue(value.aqi)">{{value.aqi}}</md-table-cell>
        <md-table-cell>{{value.lastUpdated.message}}</md-table-cell>
      </md-table-row>
    </md-table>
    <md-dialog :md-active.sync="showDialog">
      <md-dialog-title>Details</md-dialog-title>
        <div id="chart-timeline">
          <div class="toolbar">
          
            <md-button class="md-raised " id="30_seconds"
                @click="updateData('30_seconds')" :class="{'md-primary': selection==='30_seconds'}">30s</md-button>
            <md-button class="md-raised" id="one_min"
                @click="updateData('one_min')" :class="{'md-primary': selection==='one_min'}">1M</md-button>
            <md-button class="md-raised" id="5_mins"
                @click="updateData('5_mins')" :class="{'md-primary': selection==='5_mins'}">5M</md-button>
            <md-button class="md-raised" id="10_mins"
                @click="updateData('10_mins')" :class="{'md-primary': selection==='10_mins'}">10M</md-button>
            <md-button class="md-raised"  id="all"
                @click="updateData('all')" :class="{'md-primary': selection==='all'}">ALL</md-button>
          </div>
          <apexchart type="area" height="350" ref="chart" :options="chartOptions" :series="series"></apexchart>
        </div>
      <md-dialog-actions>
        <md-button class="md-primary" @click="showDialog = false">Close</md-button>
      </md-dialog-actions>
    </md-dialog>
    
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import VueApexCharts from 'vue-apexcharts';
export default {
  name: 'App',
  components: {
    apexchart: VueApexCharts,
  },
  created() {
      this.connection = new WebSocket("wss://city-ws.herokuapp.com");
      this.connection.onopen = function(event){
        console.log(event);
      }
      this.connection.onmessage = (event) => {
        console.log(event.data)
        this.handleUpdateMessage(JSON.parse(event.data));
      }
  },
  data() {
    return {
      connection:undefined,
      tableData:{},
      showDialog:false,
      series: [{
        name: "AQI",
        data: [
          [new Date(), 34], 
          [1486771200000, 43], 
          [1486857600000, 31] , 
          [1486944000000, 43], 
          [1487030400000, 33], 
          [1487116800000, 52]
        ]
      }],
      chartOptions: {
            chart: {
              id: 'area-datetime',
              type: 'area',
              height: 350,
              zoom: {
                autoScaleYaxis: true
              }
            },
            dataLabels: {
              enabled: false
            },
            markers: {
              size: 0,
              style: 'hollow',
            },
            xaxis: {
              type: 'datetime',
              min: new Date().getTime(),
              tickAmount: 6,
            },
            tooltip: {
              x: {
                format: 'dd MMM yyyy'
              }
            },
            fill: {
              type: 'gradient',
              gradient: {
                shadeIntensity: 1,
                opacityFrom: 0.7,
                opacityTo: 0.9,
                stops: [0, 100]
              }
            },
          },
      selection: '30_seconds',
    }
  },
  methods: {
    handleUpdateMessage(data){
      data.forEach(({city,aqi}) => {
        var message, seriesData;
        var aqiData = aqi.toFixed(2);
        var _self = this;
        if(_self.tableData[city] && _self.tableData[city].lastUpdated){
          seriesData = _self.tableData[city].series;
          seriesData[0].data.push([new Date(),aqiData])
          message = _self.$moment(_self.tableData[city].lastUpdated.time).fromNow();
        }
        else{
          seriesData = [{
            name: "AQI",
            data: [
              [new Date(),aqiData]
            ]
          }]
          message = _self.$moment().fromNow();
        }
        console.log(_self.tableData)
        _self.$set(_self.tableData,city,{aqi:aqiData,lastUpdated:{message:message,time: new Date()},series:seriesData})
      });
      
    },
    openChart(series){
      this.showDialog = true;
      this.series = series;
    },
    colorValue(aqi){
      return {'color':aqi<50?'Green':aqi>50 && aqi<100?'#a3c853':aqi>100 && aqi<200?'#fff835':aqi>200 && aqi<300?'#f29c33':aqi>300 && aqi<400?'#e83f33':'#af2d24'}
    },
    updateData: function(timeline) {
      this.selection = timeline
  
      switch (timeline) {
        case 'one_min':
          this.$refs.chart.zoomX(
            new Date().getTime()-60000,
            new Date().getTime()            
          )
          break
        case '30_seconds':
          this.$refs.chart.zoomX(
            new Date().getTime()-30000,
            new Date().getTime()
            
          )
          break
        case '10_mins':
          this.$refs.chart.zoomX(
            new Date().getTime()-600000,
            new Date().getTime()
          )
          break
        case '5_mins':
          this.$refs.chart.zoomX(
            new Date().getTime()-300000,
            new Date().getTime()
          )
          break
        case 'all':
          this.$refs.chart.zoomX(
            this.series[0].data[0][0],
            new Date().getTime()
          )
          break
        default:
      }
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.md-table-head-container{
  text-align: center;
}
.md-dialog-container{
  width: 70vw;
}
.md-table-row{
  cursor: pointer;
}
</style>
