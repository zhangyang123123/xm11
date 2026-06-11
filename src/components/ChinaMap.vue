<template>
  <div class="panel" :class="{ active: isActive }">
    <div class="panel-title">
      全国各省份出口热力分布
      <span class="title-sub">（单位：亿元）</span>
    </div>
    <div class="panel-content">
      <div ref="chartRef" class="chart-container"></div>
      <div v-if="!mapLoaded" class="map-loading">
        <div class="loading-spinner"></div>
        <span>地图加载中...</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as echarts from 'echarts'
import { provinceTradeData, chinaGeoJsonUrl } from '../data/mockData.js'

const props = defineProps({
  isActive: {
    type: Boolean,
    default: false
  }
})

const chartRef = ref(null)
const mapLoaded = ref(false)
let chartInstance = null
let resizeObserver = null

const initChart = () => {
  if (!chartRef.value) return

  chartInstance = echarts.init(chartRef.value)
  loadMapData()
}

const loadMapData = async () => {
  try {
    const response = await fetch(chinaGeoJsonUrl)
    const geoJson = await response.json()
    echarts.registerMap('china', geoJson)
    mapLoaded.value = true
    renderChart()
  } catch (error) {
    console.error('地图数据加载失败:', error)
  }
}

const renderChart = () => {
  if (!chartInstance) return

  const data = provinceTradeData.map(item => ({
    name: item.name.replace(/省|市|自治区|壮族|回族|维吾尔|特别行政区/g, ''),
    value: item.export,
    export: item.export,
    import: item.import,
    total: item.total,
    growth: item.growth
  }))

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
        if (!params.data) return params.name
        const d = params.data
        const growthColor = d.growth >= 0 ? '#00f5ff' : '#ff5555'
        const growthSign = d.growth >= 0 ? '+' : ''
        return `
          <div style="padding: 8px; min-width: 180px;">
            <div style="font-size: 16px; font-weight: 600; color: #00f5ff; margin-bottom: 10px; border-bottom: 1px solid rgba(0,245,255,0.3); padding-bottom: 6px;">
              ${params.name}
            </div>
            <div style="display: flex; justify-content: space-between; margin: 6px 0;">
              <span style="color: #8fb8e0;">进出口总额:</span>
              <span style="color: #e8f4ff; font-weight: 600;">${d.total.toLocaleString()} 亿元</span>
            </div>
            <div style="display: flex; justify-content: space-between; margin: 6px 0;">
              <span style="color: #8fb8e0;">出口额:</span>
              <span style="color: #00f5ff; font-weight: 600;">${d.export.toLocaleString()} 亿元</span>
            </div>
            <div style="display: flex; justify-content: space-between; margin: 6px 0;">
              <span style="color: #8fb8e0;">进口额:</span>
              <span style="color: #ff8a00; font-weight: 600;">${d.import.toLocaleString()} 亿元</span>
            </div>
            <div style="display: flex; justify-content: space-between; margin: 6px 0;">
              <span style="color: #8fb8e0;">同比增长:</span>
              <span style="color: ${growthColor}; font-weight: 600;">${growthSign}${d.growth}%</span>
            </div>
          </div>
        `
      }
    },
    visualMap: {
      min: 0,
      max: 30000,
      left: 20,
      bottom: 20,
      showLabel: true,
      text: ['高', '低'],
      textStyle: {
        color: '#8fb8e0',
        fontSize: 11
      },
      calculable: true,
      inRange: {
        color: [
          '#0a2a4a',
          '#0e4a7a',
          '#1270aa',
          '#15a0d0',
          '#00e0ff',
          '#00f5ff'
        ]
      },
      itemWidth: 12,
      itemHeight: 100
    },
    geo: {
      map: 'china',
      roam: false,
      zoom: 1.2,
      layoutCenter: ['50%', '52%'],
      layoutSize: '100%',
      label: {
        show: false
      },
      itemStyle: {
        areaColor: '#0a2a4a',
        borderColor: 'rgba(0, 245, 255, 0.3)',
        borderWidth: 1
      },
      emphasis: {
        label: {
          show: true,
          color: '#ffffff',
          fontSize: 12,
          fontWeight: 600
        },
        itemStyle: {
          areaColor: '#0088cc',
          borderColor: '#00f5ff',
          borderWidth: 2,
          shadowColor: 'rgba(0, 245, 255, 0.8)',
          shadowBlur: 20
        }
      }
    },
    series: [
      {
        name: '出口额',
        type: 'map',
        map: 'china',
        geoIndex: 0,
        animationDuration: 2000,
        animationEasing: 'elasticOut',
        data: data
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
  if (resizeObserver) resizeObserver.disconnect()
  chartInstance?.dispose()
})
</script>

<style scoped>
.map-loading {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  color: var(--text-secondary);
  font-size: 14px;
}

.loading-spinner {
  width: 36px;
  height: 36px;
  border: 3px solid rgba(0, 245, 255, 0.2);
  border-top-color: var(--cyan);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.title-sub {
  font-size: 12px;
  color: var(--text-muted);
  font-weight: 400;
  margin-left: 8px;
}
</style>
