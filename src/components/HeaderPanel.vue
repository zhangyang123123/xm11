<template>
  <div class="header-wrapper">
    <div class="header-title-bar">
      <div class="header-decor left"></div>
      <h1 class="main-title">
        <span class="title-icon">◆</span>
        中国货物贸易进出口数据可视化大屏
        <span class="title-icon">◆</span>
      </h1>
      <div class="header-info">
        <div class="time-display">
          <span class="time-icon">⏱</span>
          <span class="time-text">{{ currentTime }}</span>
        </div>
        <div class="date-display">{{ currentDate }}</div>
      </div>
      <div class="header-decor right"></div>
    </div>
    <div class="news-marquee">
      <div class="marquee-label">
        <span class="label-dot"></span>
        财经快讯
      </div>
      <div class="marquee-track">
        <div class="marquee-content" :style="{ animationDuration: marqueeDuration + 's' }">
          <span v-for="(item, index) in newsList" :key="index" class="news-item">
            <span class="news-time">[{{ item.time }}]</span>
            {{ item.content }}
          </span>
          <span v-for="(item, index) in newsList" :key="'dup-' + index" class="news-item">
            <span class="news-time">[{{ item.time }}]</span>
            {{ item.content }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { newsFlash } from '../data/mockData.js'

const currentTime = ref('')
const currentDate = ref('')
const newsList = ref(newsFlash)
const marqueeDuration = ref(60)

let timer = null

const updateTime = () => {
  const now = new Date()
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  const seconds = String(now.getSeconds()).padStart(2, '0')
  currentTime.value = `${hours}:${minutes}:${seconds}`

  const year = now.getFullYear()
  const month = String(now.getMonth() + 1).padStart(2, '0')
  const day = String(now.getDate()).padStart(2, '0')
  const weekDays = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六']
  currentDate.value = `${year}年${month}月${day}日 ${weekDays[now.getDay()]}`
}

onMounted(() => {
  updateTime()
  timer = setInterval(updateTime, 1000)
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
})
</script>

<style scoped>
.header-wrapper {
  display: flex;
  flex-direction: column;
  gap: 8px;
  height: 100%;
}

.header-title-bar {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  height: 58px;
}

.header-decor {
  position: absolute;
  top: 50%;
  width: 25%;
  height: 2px;
  transform: translateY(-50%);
  background: linear-gradient(90deg, transparent, var(--cyan), transparent);
}

.header-decor.left {
  left: 0;
}

.header-decor.right {
  right: 0;
}

.main-title {
  font-size: 32px;
  font-weight: 700;
  background: linear-gradient(180deg, #ffffff 0%, var(--cyan) 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  letter-spacing: 6px;
  text-shadow: 0 0 30px rgba(0, 245, 255, 0.5);
  white-space: nowrap;
  display: flex;
  align-items: center;
  gap: 16px;
}

.title-icon {
  color: var(--orange);
  font-size: 18px;
  -webkit-text-fill-color: var(--orange);
  animation: pulse 2s ease-in-out infinite;
}

.header-info {
  position: absolute;
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
  text-align: right;
}

.time-display {
  display: flex;
  align-items: center;
  gap: 8px;
  justify-content: flex-end;
}

.time-icon {
  color: var(--cyan);
  font-size: 16px;
}

.time-text {
  font-family: 'Courier New', monospace;
  font-size: 22px;
  font-weight: 600;
  color: var(--cyan-light);
  letter-spacing: 2px;
}

.date-display {
  font-size: 13px;
  color: var(--text-secondary);
  margin-top: 2px;
}

.news-marquee {
  display: flex;
  align-items: center;
  height: 28px;
  background: rgba(0, 136, 255, 0.08);
  border: 1px solid rgba(0, 245, 255, 0.15);
  border-radius: 2px;
  overflow: hidden;
}

.marquee-label {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 0 16px;
  height: 100%;
  background: linear-gradient(90deg, rgba(255, 138, 0, 0.2), rgba(255, 138, 0, 0.05));
  border-right: 1px solid rgba(255, 138, 0, 0.3);
  color: var(--orange);
  font-size: 13px;
  font-weight: 600;
  white-space: nowrap;
}

.label-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--orange);
  box-shadow: 0 0 10px var(--orange);
  animation: pulse 1.5s ease-in-out infinite;
}

.marquee-track {
  flex: 1;
  overflow: hidden;
  position: relative;
  height: 100%;
}

.marquee-content {
  display: flex;
  align-items: center;
  height: 100%;
  white-space: nowrap;
  animation: marquee linear infinite;
}

.news-item {
  display: inline-flex;
  align-items: center;
  padding: 0 30px;
  font-size: 13px;
  color: var(--text-secondary);
}

.news-time {
  color: var(--cyan);
  margin-right: 8px;
  font-weight: 500;
}
</style>
