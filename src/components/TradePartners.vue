<template>
  <div class="panel" :class="{ active: isActive }">
    <div class="panel-title">
      主要贸易伙伴排行
      <span class="title-sub">Top 10（单位：亿元）</span>
    </div>
    <div class="panel-content">
      <div ref="chartRef" class="chart-container"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as echarts from 'echarts'
import { tradePartners } from '../data/mockData.js'

const props = defineProps({
  isActive: {
    type: Boolean,
    default: false
  }
})

const chartRef = ref(null)
let chartInstance = null

const initChart = () => {
  if (!chartRef.value) return

  chartInstance = echarts.init(chartRef.value)
  renderChart()
}

const renderChart = () => {
  if (!chartInstance) return

  const data = [...tradePartners].slice(0, 10).reverse()
  const maxValue = Math.max(...data.map(d => d.value))

  const option = {
    grid: {
      left: 80,
      right: 60,
      top: 10,
      bottom: 10,
      containLabel: false
    },
    xAxis: {
      type: 'value',
      show: false,
      max: maxValue * 1.15
    },
    yAxis: {
      type: 'category',
      data: data.map(d => d.name),
      axisLine: {
        show: false
      },
      axisTick: {
        show: false
      },
      axisLabel: {
        color: '#e8f4ff',
        fontSize: 12,
        fontWeight: 500,
        margin: 12
      }
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow'
      },
      backgroundColor: 'rgba(10, 22, 40, 0.95)',
      borderColor: 'rgba(0, 245, 255, 0.5)',
      borderWidth: 1,
      textStyle: {
        color: '#e8f4ff',
        fontSize: 13
      },
      formatter: (params) => {
        const item = params[0]
        const partner = tradePartners.find(p => p.name === item.name)
        const growthColor = partner.growth >= 0 ? '#00f5ff' : '#ff5555'
        const growthSign = partner.growth >= 0 ? '+' : ''
        return `
          <div style="padding: 8px;">
            <div style="font-size: 14px; font-weight: 600; color: #00f5ff; margin-bottom: 8px;">
              ${item.name}
            </div>
            <div style="display: flex; justify-content: space-between; gap: 20px; margin: 4px 0;">
              <span style="color: #8fb8e0;">贸易额:</span>
              <span style="color: #e8f4ff; font-weight: 600;">${item.value.toLocaleString()} 亿元</span>
            </div>
            <div style="display: flex; justify-content: space-between; gap: 20px; margin: 4px 0;">
              <span style="color: #8fb8e0;">同比增长:</span>
              <span style="color: ${growthColor}; font-weight: 600;">${growthSign}${partner.growth}%</span>
            </div>
          </div>
        `
      }
    },
    series: [
      {
        name: '贸易额',
        type: 'bar',
        data: data.map((d, index) => ({
          value: d.value,
          itemStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 1, 0, [
              { offset: 0, color: index >= 7 ? '#ff8a00' : 'rgba(0, 136, 255, 0.3)' },
              { offset: 1, color: index >= 7 ? '#00f5ff' : '#00f5ff' }
            ]),
            borderRadius: [0, 4, 4, 0]
          }
        })),
        barWidth: 12,
        label: {
          show: true,
          position: 'right',
          color: '#e8f4ff',
          fontSize: 12,
          fontWeight: 600,
          formatter: (params) => params.value.toLocaleString()
        },
        showBackground: true,
        backgroundStyle: {
          color: 'rgba(0, 245, 255, 0.06)',
          borderRadius: [0, 4, 4, 0]
        },
        animationDelay: (idx) => idx * 100,
        animationDuration: 1500,
        animationEasing: 'elasticOut'
      }
    ]
  }

  chartInstance.setOption(option)
}

const handleResize = () => {
  chartInstance?.resize()
}

watch(() => props.isActive, (val) => {
  if (val && chartInstance) {
    setTimeout(() => {
      chartInstance?.resize()
    }, 100)
  }
})

onMounted(() => {
  initChart()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  chartInstance?.dispose()
})
</script>

<style scoped>
.title-sub {
  font-size: 12px;
  color: var(--text-muted);
  font-weight: 400;
  margin-left: 8px;
}
</style>
