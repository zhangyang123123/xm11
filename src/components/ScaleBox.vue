<template>
  <div class="scale-wrapper" ref="wrapperRef">
    <div
      class="scale-content"
      :style="scaleStyle"
      ref="contentRef"
    >
      <slot></slot>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted, computed } from 'vue'

const props = defineProps({
  width: {
    type: Number,
    default: 1920
  },
  height: {
    type: Number,
    default: 1080
  }
})

const wrapperRef = ref(null)
const contentRef = ref(null)
const scale = reactive({ x: 1, y: 1 })

const scaleStyle = computed(() => ({
  width: props.width + 'px',
  height: props.height + 'px',
  transform: `scale(${scale.x}, ${scale.y})`,
  transformOrigin: 'left top'
}))

const calcScale = () => {
  if (!wrapperRef.value) return

  const wrapperWidth = window.innerWidth
  const wrapperHeight = window.innerHeight

  const scaleX = wrapperWidth / props.width
  const scaleY = wrapperHeight / props.height

  const scaleValue = Math.min(scaleX, scaleY)

  scale.x = scaleValue
  scale.y = scaleValue

  const contentEl = contentRef.value
  if (contentEl) {
    const scaledWidth = props.width * scaleValue
    const scaledHeight = props.height * scaleValue
    const offsetLeft = (wrapperWidth - scaledWidth) / 2
    const offsetTop = (wrapperHeight - scaledHeight) / 2
    contentEl.style.marginLeft = offsetLeft + 'px'
    contentEl.style.marginTop = offsetTop + 'px'
  }
}

let resizeTimer = null

const handleResize = () => {
  if (resizeTimer) {
    clearTimeout(resizeTimer)
  }
  resizeTimer = setTimeout(() => {
    calcScale()
    window.dispatchEvent(new Event('resize'))
  }, 100)
}

onMounted(() => {
  calcScale()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  if (resizeTimer) clearTimeout(resizeTimer)
})
</script>

<style scoped>
.scale-wrapper {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: #000;
}

.scale-content {
  position: relative;
  will-change: transform;
}
</style>
