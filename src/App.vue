<template>
  <div class="dashboard-container">
    <header class="dashboard-header">
      <HeaderPanel />
    </header>

    <main class="dashboard-main">
      <div class="col-left">
        <ChinaMap :is-active="activePanel === 0" class="flex-1" />
      </div>

      <div class="col-center">
        <SummaryCards :is-active="activePanel === 1" />
      </div>

      <div class="col-right">
        <TradePartners :is-active="activePanel === 2" class="flex-1" />
      </div>
    </main>

    <footer class="dashboard-footer">
      <MonthlyChart :is-active="activePanel === 3" style="flex: 1.5" />
      <CategoryPie :is-active="activePanel === 4" style="flex: 1" />
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import HeaderPanel from './components/HeaderPanel.vue'
import ChinaMap from './components/ChinaMap.vue'
import SummaryCards from './components/SummaryCards.vue'
import TradePartners from './components/TradePartners.vue'
import MonthlyChart from './components/MonthlyChart.vue'
import CategoryPie from './components/CategoryPie.vue'

const activePanel = ref(-1)
const PANEL_COUNT = 5
const HIGHLIGHT_INTERVAL = 8000

let highlightTimer = null

const startHighlightCycle = () => {
  activePanel.value = 0
  highlightTimer = setInterval(() => {
    activePanel.value = (activePanel.value + 1) % PANEL_COUNT
  }, HIGHLIGHT_INTERVAL)
}

onMounted(() => {
  setTimeout(startHighlightCycle, 2000)
})

onUnmounted(() => {
  if (highlightTimer) {
    clearInterval(highlightTimer)
  }
})
</script>

<style>
.flex-1 {
  flex: 1;
}
</style>
