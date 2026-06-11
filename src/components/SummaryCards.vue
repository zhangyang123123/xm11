<template>
  <div class="panel summary-panel" :class="{ active: isActive }">
    <div class="panel-title">
      核心指标概览
      <span class="period-badge">{{ summary.period }}</span>
    </div>
    <div class="panel-content">
      <div class="summary-grid">
        <div class="summary-card primary">
          <div class="card-header">
            <span class="card-icon">📊</span>
            <span class="card-title">进出口总值</span>
          </div>
          <div class="card-value">
            <span class="number-cyan">{{ animatedValues.total }}</span>
            <span class="unit">{{ summary.totalTradeValueUnit }}</span>
          </div>
          <div class="card-growth positive" v-if="summary.totalTradeGrowth >= 0">
            <span class="growth-arrow">▲</span>
            同比增长 <span class="growth-value">{{ animatedValues.totalGrowth }}%</span>
          </div>
          <div class="card-growth negative" v-else>
            <span class="growth-arrow">▼</span>
            同比下降 <span class="growth-value">{{ Math.abs(animatedValues.totalGrowth) }}%</span>
          </div>
        </div>

        <div class="summary-card export">
          <div class="card-header">
            <span class="card-icon">📤</span>
            <span class="card-title">出口总值</span>
          </div>
          <div class="card-value">
            <span class="number-cyan">{{ animatedValues.export }}</span>
            <span class="unit">{{ summary.exportValueUnit }}</span>
          </div>
          <div class="card-growth positive" v-if="summary.exportGrowth >= 0">
            <span class="growth-arrow">▲</span>
            同比增长 <span class="growth-value">{{ animatedValues.exportGrowth }}%</span>
          </div>
          <div class="card-growth negative" v-else>
            <span class="growth-arrow">▼</span>
            同比下降 <span class="growth-value">{{ Math.abs(animatedValues.exportGrowth) }}%</span>
          </div>
        </div>

        <div class="summary-card import">
          <div class="card-header">
            <span class="card-icon">📥</span>
            <span class="card-title">进口总值</span>
          </div>
          <div class="card-value">
            <span class="number-orange">{{ animatedValues.import }}</span>
            <span class="unit">{{ summary.importValueUnit }}</span>
          </div>
          <div class="card-growth positive" v-if="summary.importGrowth >= 0">
            <span class="growth-arrow">▲</span>
            同比增长 <span class="growth-value">{{ animatedValues.importGrowth }}%</span>
          </div>
          <div class="card-growth negative" v-else>
            <span class="growth-arrow">▼</span>
            同比下降 <span class="growth-value">{{ Math.abs(animatedValues.importGrowth) }}%</span>
          </div>
        </div>

        <div class="summary-card surplus">
          <div class="card-header">
            <span class="card-icon">💹</span>
            <span class="card-title">贸易顺差</span>
          </div>
          <div class="card-value">
            <span class="number-green">{{ animatedValues.surplus }}</span>
            <span class="unit">{{ summary.surplusValueUnit }}</span>
          </div>
          <div class="card-growth positive" v-if="summary.surplusGrowth >= 0">
            <span class="growth-arrow">▲</span>
            同比增长 <span class="growth-value">{{ animatedValues.surplusGrowth }}%</span>
          </div>
          <div class="card-growth negative" v-else>
            <span class="growth-arrow">▼</span>
            同比下降 <span class="growth-value">{{ Math.abs(animatedValues.surplusGrowth) }}%</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, reactive } from 'vue'
import { summaryData } from '../data/mockData.js'

const props = defineProps({
  isActive: {
    type: Boolean,
    default: false
  }
})

const summary = ref(summaryData)

const animatedValues = reactive({
  total: '0',
  totalGrowth: '0.0',
  export: '0',
  exportGrowth: '0.0',
  import: '0',
  importGrowth: '0.0',
  surplus: '0',
  surplusGrowth: '0.0'
})

const animateNumber = (target, key, isFloat = false, duration = 2000) => {
  const startTime = performance.now()
  const startValue = 0

  const step = (currentTime) => {
    const elapsed = currentTime - startTime
    const progress = Math.min(elapsed / duration, 1)
    const easeProgress = 1 - Math.pow(1 - progress, 3)
    const currentValue = startValue + (target - startValue) * easeProgress

    if (isFloat) {
      animatedValues[key] = currentValue.toFixed(1)
    } else {
      animatedValues[key] = Math.floor(currentValue).toLocaleString()
    }

    if (progress < 1) {
      requestAnimationFrame(step)
    }
  }

  requestAnimationFrame(step)
}

const startAnimation = () => {
  animateNumber(summary.value.totalTradeValue, 'total')
  animateNumber(summary.value.totalTradeGrowth, 'totalGrowth', true)
  animateNumber(summary.value.exportValue, 'export')
  animateNumber(summary.value.exportGrowth, 'exportGrowth', true)
  animateNumber(summary.value.importValue, 'import')
  animateNumber(summary.value.importGrowth, 'importGrowth', true)
  animateNumber(summary.value.surplusValue, 'surplus')
  animateNumber(summary.value.surplusGrowth, 'surplusGrowth', true)
}

watch(() => props.isActive, (val) => {
  if (val) {
    startAnimation()
  }
})

onMounted(() => {
  setTimeout(startAnimation, 500)
})
</script>

<style scoped>
.summary-panel {
  flex: 1;
}

.period-badge {
  margin-left: auto;
  font-size: 12px;
  font-weight: 400;
  color: var(--orange);
  background: rgba(255, 138, 0, 0.1);
  padding: 2px 10px;
  border-radius: 10px;
  border: 1px solid rgba(255, 138, 0, 0.3);
}

.summary-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: 1fr 1fr;
  gap: 16px;
  height: 100%;
}

.summary-card {
  position: relative;
  background: linear-gradient(135deg, rgba(0, 136, 255, 0.05) 0%, rgba(0, 245, 255, 0.02) 100%);
  border: 1px solid var(--border-color);
  border-radius: 4px;
  padding: 16px 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  overflow: hidden;
  transition: all 0.3s ease;
}

.summary-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 3px;
  height: 100%;
  background: var(--gradient-cyan);
  opacity: 0.6;
}

.summary-card.export::before {
  background: var(--gradient-cyan);
}

.summary-card.import::before {
  background: var(--gradient-orange);
}

.summary-card.surplus::before {
  background: linear-gradient(135deg, #00ff88 0%, #00cc6a 100%);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.card-icon {
  font-size: 16px;
}

.card-title {
  font-size: 13px;
  color: var(--text-secondary);
  font-weight: 500;
}

.card-value {
  display: flex;
  align-items: baseline;
  gap: 6px;
  margin-bottom: 6px;
}

.number-cyan,
.number-orange,
.number-green {
  font-family: 'Courier New', monospace;
  font-size: 32px;
  font-weight: 700;
  letter-spacing: 1px;
}

.number-cyan {
  color: var(--cyan);
  text-shadow: 0 0 20px rgba(0, 245, 255, 0.5);
}

.number-orange {
  color: var(--orange);
  text-shadow: 0 0 20px rgba(255, 138, 0, 0.5);
}

.number-green {
  color: #00ff88;
  text-shadow: 0 0 20px rgba(0, 255, 136, 0.5);
}

.unit {
  font-size: 14px;
  color: var(--text-muted);
}

.card-growth {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 12px;
}

.card-growth.positive {
  color: #00ff88;
}

.card-growth.negative {
  color: #ff5555;
}

.growth-arrow {
  font-size: 10px;
}

.growth-value {
  font-weight: 600;
  font-family: 'Courier New', monospace;
}

.summary-card.primary {
  grid-column: span 2;
  background: linear-gradient(135deg, rgba(0, 245, 255, 0.1) 0%, rgba(0, 136, 255, 0.05) 100%);
  border-color: rgba(0, 245, 255, 0.3);
}

.summary-card.primary .number-cyan {
  font-size: 42px;
}

.summary-card.primary .card-title {
  font-size: 14px;
}
</style>
