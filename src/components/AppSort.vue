<template>
  <div :id="id" :class="className" :style="{height:height,width:width}" />
</template>

<script>
import echarts from 'echarts'
import resize from './mixins/resize'

import { searchApp } from '@/api/user'
import { formatFileSize } from '@/utils/myUtils'
import { refreshData } from '@/utils/myUtils'
import themeJson from '@/assets/json/theme.json'

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'paidChart'
    },
    id: {
      type: String,
      default: 'paidChart'
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
        legendData: [],
        seriesData: []
      },
      darkTheme: themeJson.darkTheme,
      brightTheme: themeJson.brightTheme,
      aggs: {
        'type_total': {
          'terms': {
            'size': 8,
            'field': 'application_name'
          }, 'aggs': {
            'byte_total': {
              'sum': {
                'field': 'nbytes'
              }
            }
          }
        }
      }
    }
  },
  computed: {
    reqDate() {
      return this.$store.getters.date
    },
    time() {
      // 获取刷新时间
      return this.$store.getters.time
    },
    theme() {
      return this.$store.getters.theme
    },
    themeStyle() {
      if (this.theme === 'dark') {
        return this.darkTheme
      } else {
        return this.brightTheme
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
    this.getData()
    this.$nextTick(() => {
      this.initChart()
    })
    refreshData(this.getData, this.reqDate, this.aggs, this.time)
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    getData() {
      searchApp(this.reqDate, 0, this.aggs).then(res => {
        var result = res.aggregations.type_total.buckets
        result.forEach((e, i) => {
          this.chartData.legendData.splice(i, 1, e.key)
          this.chartData.seriesData.splice(i, 1,
            {
              name: e.key,
              value: e.byte_total.value
            }
          )
        })
      })
    },
    initChart() {
      this.chart = echarts.init(document.getElementById(this.id))
      this.setOptions(this.chartData, true)
    },

    setOptions({ legendData, seriesData }) {
      this.chart.setOption({
        // color: ['#A6E3A1', '#BB3884', '#E7E6D4', '#F9E79F', '#91BAA0', '#F1B9BA', '#BCDEF7', '#C1C6D9'],
        color: ['rgb(0,192,239)', 'rgb(243,156,18)', 'rgb(0,166,90)', 'rgb(221,75,57)', 'rgb(96,92,168)',
          'rgb( 240,18,190)', 'rgb(119,119,199)', 'rgb(0,115,183)', 'rgb(255,133,27)', 'rgb(1,255,112)', 'rgb(0,31,63)'],
        backgroundColor: this.themeStyle.chartColor.bgColor,
        title: [{
          text: '[DSOC]应用分类',
          left: 'center',
          y: '10',
          textStyle: {
            color: this.themeStyle.chartColor.fontColor,
            fontSize: '12'
          }
        }],
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)',
          color: 'white'
        },
        formatter: function(val) { return val.length > 12 ? val.substring(0, 12) + '...' : val },
        legend: {
          textStyle: {
            color: this.themeStyle.chartColor.fontColor,
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
                formatter: function(params) {
                  return params.name + ':' + formatFileSize(params.value)
                },
                borderColor: '#aaa',
                fontSize: 13
              }
            },
            data: seriesData
          }
        ]
      })
    }
  }
}
// }
</script>
