<!--
 ADempiere-Vue (Frontend) for ADempiere ERP & CRM Smart Business Solution
 Copyright (C) 2017-Present E.R.P. Consultores y Asociados, C.A.
 Contributor(s): Yamel Senih ysenih@erpya.com www.erpya.com
 This program is free software: you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation, either version 3 of the License, or
 (at your option) any later version.

 This program is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.

 You should have received a copy of the GNU General Public License
 along with this program.  If not, see <https:www.gnu.org/licenses/>.
-->
<template>
  <div :class="className" :style="{height:height,width:width}" />
</template>

<script>
import * as echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from './mixins/resize'
import { getMetrics } from '@/api/ADempiere/dashboard/chart'

export default {
  mixins: [resize],
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
      default: '300px'
    },
    metadata: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      chart: null
    }
  },
  mounted() {
    this.unsubscribe = this.subscribeChanges()
    this.$nextTick(() => {
      this.initChart()
    })
  },
  beforeDestroy() {
    this.unsubscribe()
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    subscribeChanges() {
      return this.$store.subscribe((mutation, state) => {
        if (mutation.type === 'notifyDashboardRefresh') {
          this.initChart()
        }
      })
    },
    initChart() {
      this.chart = echarts.init(this.$el, 'macarons')
      this.chart.showLoading()
      if (!this.isEmptyValue(this.metadata.actions)) {
        this.$store.dispatch('metrics', {
          id: this.metadata.id,
          tableName: this.metadata.tableName,
          recordId: this.metadata.recordId,
          recordUuid: this.metadata.recordUuid
        })
          .then(response => {
            this.loadChartMetrics(response)
          })
          .catch(error => {
            console.warn(`Error getting Metrics: ${error.message}. Code: ${error.code}.`)
          })
        return
      }
      getMetrics({
        id: this.metadata.id
      })
        .then(metrics => {
          this.loadChartMetrics(metrics)
        })
        .catch(error => {
          console.warn(`Error getting Metrics: ${error.message}. Code: ${error.code}.`)
        })
    },
    loadChartMetrics(metrics) {
      const xAxisValues = []
      let seriesToShow = []
      let legendToShow = []
      if (!this.isEmptyValue(metrics.series)) {
        if (metrics.series.length > 0) {
          metrics.series.forEach(serie => {
            serie.data_set.forEach(set => {
              if (!xAxisValues.find(value => value === set.name)) {
                xAxisValues.push(set.name)
              }
            })
          })
        }
        seriesToShow = metrics.series.map(serie => {
          return {
            name: serie.name,
            type: 'bar',
            data: serie.data_set.map(set => set.value),
            animationDelay: function(idx) {
              return idx * 10
            }
          }
        })
        legendToShow = metrics.series.map(serie => serie.name)
      }
      this.chart.setOption({
        tooltip: {
          backgroundColor: '#FFF',
          trigger: 'axis',
          axisPointer: {
            type: 'shadow'
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        toolbox: {
          // y: 'bottom',
          feature: {
            magicType: {
              type: ['stack', 'tiled']
            },
            dataView: {},
            saveAsImage: {
              pixelRatio: 2
            }
          }
        },
        xAxis: [{
          data: xAxisValues,
          type: 'category',
          splitLine: {
            show: false
          }
        }],
        yAxis: {
        },
        legend: {
          data: legendToShow
        },
        series: seriesToShow,
        animationEasing: 'elasticOut',
        animationDelayUpdate: function(idx) {
          return idx * 5
        }
      })
      this.chart.hideLoading()
    }
  }
}
</script>
