<template>
  <div class="panel" :class="{ active: isActive }">
    <div class="panel-title">
      主要出口商品类别占比
      <span class="title-sub">（单位：%）</span>
    </div>
    <div class="panel-content">
      <div ref="chartRef" class="chart-container"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as echarts from 'echarts'
import { exportCategories } from '../data/mockData.js'

const props = defineProps({
  isActive: {
    type: Boolean,
    default: false
  }
})

const chartRef = ref(null)
let chartInstance = null

const colors = [
  '#00f5ff',
  '#00a8ff',
  '#ff8a00',
  '#00ff88',
  '#ff5577',
  '#aa66ff',
  '#ffcc00',
  '#5a7ca0'
]

const initChart = () => {
  if (!chartRef.value) return

  chartInstance = echarts.init(chartRef.value)
  renderChart()
}

const renderChart = () => {
  if (!chartInstance) return

  const option = {
    tooltip: {
      trigger: 'item',
      backgroundColor: 'rgba(10, 22, 40, 0.95)',
      borderColor: 'rgba(0, 245, 255, 0.5)',
      borderWidth: 1,
      textStyle: {
        color: '#e8f4ff',
        fontSize: 13
      },
      formatter: (params) => {
        return `
          <div style="padding: 8px;">
            <div style="font-size: 14px; font-weight: 600; color: #00f5ff; margin-bottom: 8px;">
              ${params.name}
            </div>
            <div style="display: flex; justify-content: space-between; gap: 20px; margin: 4px 0;">
              <span style="color: #8fb8e0;">占比:</span>
              <span style="color: #e8f4ff; font-weight: 600;">${params.value}%</span>
            </div>
            <div style="display: flex; justify-content: space-between; gap: 20px; margin: 4px 0;">
              <span style="color: #8fb8e0;">出口额:</span>
              <span style="color: #ff8a00; font-weight: 600;">${params.data.amount.toLocaleString()} 亿元</span>
            </div>
          </div>
        `
      }
    },
    legend: {
      orient: 'vertical',
      right: 10,
      top: 'center',
      textStyle: {
        color: '#8fb8e0',
        fontSize: 12
      },
      itemWidth: 10,
      itemHeight: 10,
      itemGap: 10,
      formatter: (name) => {
        const item = exportCategories.find(c => c.name === name)
        return item ? `${name}  ${item.value}%` : name
      }
    },
    series: [
      {
        name: '出口类别',
        type: 'pie',
        radius: ['45%', '72%'],
        center: ['35%', '50%'],
        avoidLabelOverlap: true,
        itemStyle: {
          borderColor: '#0a1628',
          borderWidth: 3
        },
        label: {
          show: true,
          position: 'outside',
          color: '#8fb8e0',
          fontSize: 11,
          formatter: '{b}\n{d}%',
          lineHeight: 16
        },
        labelLine: {
          show: true,
          length: 8,
          length2: 8,
          lineStyle: {
            color: 'rgba(0, 245, 255, 0.3)'
          }
        },
        emphasis: {
          label: {
            show: true,
            fontSize: 13,
            fontWeight: 'bold',
            color: '#00f5ff'
          },
          itemStyle: {
            shadowBlur: 20,
            shadowColor: 'rgba(0, 245, 255, 0.6)'
          }
        },
        data: exportCategories.map((item, index) => ({
          value: item.value,
          name: item.name,
          amount: item.amount,
          itemStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 1, 1, [
              { offset: 0, color: colors[index] },
              { offset: 1, color: colors[index] + '88' }
            ])
          }
        })),
        animationType: 'scale',
        animationEasing: 'elasticOut',
        animationDelay: (idx) => idx * 100,
        animationDuration: 1500
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
