<template>
  <div :id="id" :class="className" :style="{height:height,width:width}" />
</template>

<script>
import resize from '../mixins/resize'
import * as echarts from 'echarts'

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'BarChart'
    },
    id: {
      type: String,
      default: 'BarChart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '400px'
    }
  },
  data() {
    return {
      chart: null,
      chartData: {
        legendData: ['1', '2', '3'],
        seriesData: [1, 3, 2]
      }
    }
  },
  watch: {
    chartData: {
      deep: true,
      handler(val) {
        this.setOptions(val, true)
      }
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initChart()
    })
    setTimeout(() => {
      this.chartData.seriesData = [1, 3, 7]
    }, 1000)
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    initChart() {
      this.chart = echarts.init(document.getElementById(this.id))
      this.setOptions(this.chartData, true)
    },

    setOptions({
      legendData,
      seriesData
    }) {
      this.chart.setOption(Object.assign({
        // color: ['#A6E3A1', '#BB3884', '#E7E6D4', '#F9E79F', '#91BAA0', '#F1B9BA', '#BCDEF7', '#C1C6D9'],
        color: ['rgb(0,192,239)', 'rgb(243,156,18)', 'rgb(0,166,90)', 'rgb(221,75,57)', 'rgb(96,92,168)',
          'rgb( 240,18,190)', 'rgb(119,119,199)', 'rgb(0,115,183)', 'rgb(255,133,27)', 'rgb(1,255,112)', 'rgb(0,31,63)'],
        // backgroundColor: this.themeStyle.chartColor.bgColor,
        backgroundColor: '#eee',
        title: [{
          text: '[DSOC]应用分类',
          left: 'center',
          y: '10',
          textStyle: {
            // color: this.themeStyle.chartColor.fontColor,
            color: 'white',
            fontSize: '12'
          }
        }],
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)',
          color: 'white'
        },
        formatter(val) { return val.length > 12 ? `${val.substring(0, 12)}...` : val },
        legend: {
          textStyle: {
            // color: this.themeStyle.chartColor.fontColor,
            color: 'white',
            fontSize: '12'
          },
          orient: 'vertical',
          x: 'left',
          y: 'top',
          itemWidth: 14,
          itemHeight: 14,
          data: legendData
        },
        series: [
          {
            name: '应用分类',
            center: ['60%', '60%'],
            type: 'pie',
            radius: ['45%', '70%'],
            label: {
              normal: {
                // formatter(params) {
                //   return `${params.name}:${formatFileSize(params.value)}`;
                // },
                borderColor: '#aaa',
                fontSize: 13
              }
            },
            data: seriesData.map((e, i) => {
              return {
                value: e,
                name: legendData[i]
              }
            })
          }
        ]
      }, {
        title: [{
          text: '[DSOC]123',
          left: 'center',
          y: '10',
          textStyle: {
            // color: this.themeStyle.chartColor.fontColor,
            color: 'white',
            fontSize: '16'
          }
        }]
      }))
    }
  }
}
</script>
