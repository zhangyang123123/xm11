<template>
  <div class="panel" :class="{ active: isActive }">
    <div class="panel-title">
      月度进出口趋势
      <span class="title-sub">（单位：亿元）</span>
      <div class="tab-switcher">
        <span
          v-for="tab in tabs"
          :key="tab.value"
          class="tab-item"
          :class="{ active: activeTab === tab.value }"
          @click="activeTab = tab.value"
        >
          {{ tab.label }}
        </span>
      </div>
    </div>
    <div class="panel-content">
      <div ref="chartRef" class="chart-container"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, computed } from 'vue'
import * as echarts from 'echarts'
import { monthlyTradeData } from '../data/mockData.js'

const props = defineProps({
  isActive: {
    type: Boolean,
    default: false
  }
})

const chartRef = ref(null)
let chartInstance = null

const tabs = [
  { label: '近3月', value: 3 },
  { label: '近6月', value: 6 },
  { label: '近12月', value: 12 }
]

const activeTab = ref(12)

const filteredData = computed(() => {
  return monthlyTradeData.slice(-activeTab.value)
})

const initChart = () => {
  if (!chartRef.value) return

  chartInstance = echarts.init(chartRef.value)
  renderChart()
}

const renderChart = () => {
  if (!chartInstance) return

  const data = filteredData.value

  const option = {
    legend: {
      data: ['出口', '进口', '贸易顺差'],
      top: 0,
      right: 10,
      textStyle: {
        color: '#8fb8e0',
        fontSize: 12
      },
      itemWidth: 16,
      itemHeight: 8,
      itemGap: 20
    },
    grid: {
      left: 50,
      right: 20,
      top: 40,
      bottom: 30
    },
    tooltip: {
      trigger: 'axis',
      backgroundColor: 'rgba(10, 22, 40, 0.95)',
      borderColor: 'rgba(0, 245, 255, 0.5)',
      borderWidth: 1,
      textStyle: {
        color: '#e8f4ff',
        fontSize: 13
      },
      axisPointer: {
        type: 'cross',
        lineStyle: {
          color: 'rgba(0, 245, 255, 0.3)'
        },
        crossStyle: {
          color: 'rgba(0, 245, 255, 0.3)'
        }
      }
    },
    xAxis: {
      type: 'category',
      data: data.map(d => d.month),
      axisLine: {
        lineStyle: {
          color: 'rgba(0, 245, 255, 0.2)'
        }
      },
      axisLabel: {
        color: '#8fb8e0',
        fontSize: 11
      },
      axisTick: {
        show: false
      }
    },
    yAxis: {
      type: 'value',
      splitLine: {
        lineStyle: {
          color: 'rgba(0, 245, 255, 0.08)',
          type: 'dashed'
        }
      },
      axisLine: {
        show: false
      },
      axisLabel: {
        color: '#8fb8e0',
        fontSize: 11
      },
      axisTick: {
        show: false
      }
    },
    series: [
      {
        name: '出口',
        type: 'line',
        data: data.map(d => d.export),
        smooth: true,
        symbol: 'circle',
        symbolSize: 6,
        lineStyle: {
          width: 3,
          color: '#00f5ff',
          shadowColor: 'rgba(0, 245, 255, 0.5)',
          shadowBlur: 10
        },
        itemStyle: {
          color: '#00f5ff',
          borderColor: '#fff',
          borderWidth: 2
        },
        areaStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(0, 245, 255, 0.3)' },
            { offset: 1, color: 'rgba(0, 245, 255, 0.02)' }
          ])
        },
        animationDuration: 2000,
        animationEasing: 'cubicOut'
      },
      {
        name: '进口',
        type: 'line',
        data: data.map(d => d.import),
        smooth: true,
        symbol: 'circle',
        symbolSize: 6,
        lineStyle: {
          width: 3,
          color: '#ff8a00',
          shadowColor: 'rgba(255, 138, 0, 0.5)',
          shadowBlur: 10
        },
        itemStyle: {
          color: '#ff8a00',
          borderColor: '#fff',
          borderWidth: 2
        },
        areaStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(255, 138, 0, 0.25)' },
            { offset: 1, color: 'rgba(255, 138, 0, 0.02)' }
          ])
        },
        animationDuration: 2000,
        animationDelay: 200,
        animationEasing: 'cubicOut'
      },
      {
        name: '贸易顺差',
        type: 'line',
        data: data.map(d => d.surplus),
        smooth: true,
        symbol: 'circle',
        symbolSize: 6,
        lineStyle: {
          width: 3,
          color: '#00ff88',
          shadowColor: 'rgba(0, 255, 136, 0.5)',
          shadowBlur: 10,
          type: 'dashed'
        },
        itemStyle: {
          color: '#00ff88',
          borderColor: '#fff',
          borderWidth: 2
        },
        animationDuration: 2000,
        animationDelay: 400,
        animationEasing: 'cubicOut'
      }
    ]
  }

  chartInstance.setOption(option, true)
}

const handleResize = () => {
  chartInstance?.resize()
}

watch(activeTab, () => {
  renderChart()
})

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

.tab-switcher {
  margin-left: auto;
  display: flex;
  gap: 6px;
}

.tab-item {
  padding: 3px 12px;
  font-size: 12px;
  color: var(--text-secondary);
  background: rgba(0, 245, 255, 0.05);
  border: 1px solid rgba(0, 245, 255, 0.2);
  border-radius: 2px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 400;
}

.tab-item:hover {
  color: var(--cyan-light);
  border-color: rgba(0, 245, 255, 0.4);
}

.tab-item.active {
  color: #fff;
  background: linear-gradient(135deg, rgba(0, 245, 255, 0.3), rgba(0, 136, 255, 0.3));
  border-color: var(--cyan);
  box-shadow: 0 0 10px rgba(0, 245, 255, 0.3);
}
</style>
