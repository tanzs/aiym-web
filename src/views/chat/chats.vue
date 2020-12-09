<template>
       <div :class="className" :style="{height:height,width:width}" id="moCharts"/>
</template>

<script>
import echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import { debounce } from '@/utils'
export default {
  props: {
    className: {
      type: String,
      default: 'chart'
     },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '500px'
    },
    autoResize: {
      type: Boolean,
      default: true
    },
    //index页面type变量定义
    type:{
      type: String,
      required: true,
      default:'1',
    }
  },
  data() {
    return {
      chart: null,
      sidebarElm: null,
      //定义echarts字段，接收可变值
      chartData: {
        days:[],
        users:[]
      },
    }
  },
 //监听index传过来的type字段
  watch:{
      type:{
         handler(newValue,oldValue){
            this.type=newValue;
            this.initChart();
          }
      }

    },
  mounted() {
    //初始化echart数据
    this.initChart();

    if (this.autoResize) {
      this.__resizeHandler = debounce(() => {
        if (this.chart) {
          this.chart.resize()
        }
      }, 100)
       let myChart = document.getElementById(('moCharts'));
       myChart.style.width = window.innerWidth - 290 + 'px';
      window.addEventListener('resize', this.__resizeHandler)
    }

    // 监听侧边栏的变化
    this.sidebarElm = document.getElementsByClassName('sidebar-container')[0]
    this.sidebarElm && this.sidebarElm.addEventListener('transitionend', this.sidebarResizeHandler)
  },
  destroyed(){
       clearInterval(this.myInterval)
   },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    if (this.autoResize) {
      window.removeEventListener('resize', this.__resizeHandler)
    }

    this.sidebarElm && this.sidebarElm.removeEventListener('transitionend', this.sidebarResizeHandler)

    this.chart.dispose()
    this.chart = null
  },
  methods: {
    sidebarResizeHandler(e) {
      if (e.propertyName === 'width') {
        this.__resizeHandler()
      }
    },
    initChart() {
    // 数据修改
        this.chartData.days=["2020-12-01","2020-12-02","2020-12-03","2020-12-04","2020-12-05","2020-12-06","2020-12-07","2020-12-08","2020-12-09"];
        if(this.type==1){
            this.chartData.users=[19,29,30,45,40,47,50,53,57];
        }else{
            this.chartData.users=[10,19,20,25,30,37,40,43,47];
        }
        //渲染echarts组件
        this.chart = echarts.init(this.$el, 'macarons')
        this.chart.setOption({
             //鼠标移上去显示数值工具
             tooltip: {
                  trigger: 'axis',
                    axisPointer: {
                      type: 'cross',
                      crossStyle: {
                        color: '#999'
                    }
                 }
                },
              //x轴
             xAxis: {
                type: 'category',
                data: this.chartData.days,
             },
             yAxis: {
                type: 'value'
             },
             legend: {
                 data: ['新增用户']
              },
             series: [{
                 name:'新增用户',
                 data: this.chartData.users,
                 type: 'line'
                  }]
             })
            }
  }
  }
</script>

