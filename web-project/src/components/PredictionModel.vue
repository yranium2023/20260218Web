<template>
  <div class="page">
    <div class="card">
      <header class="card-header card-header-sticky">
        <div class="header-badge">预测模型</div>
        <h1>成虫诱捕量与林间幼虫量预测模型</h1>
        <p class="subtitle">基于二次回归模型的交互式可视化分析</p>
      </header>

      <main class="card-body">
      <section class="model-section">
        <h2><span class="section-num">1</span> 模型公式</h2>
        <div class="formula-box">
          <p class="formula">
            <em>y</em> = <span class="coef">0.2633</span><em>x</em>²
            <span class="coef">− 6.9109</span><em>x</em>
            <span class="coef">+ 51.8272</span>
          </p>
          <p class="r2">决定系数 <em>R</em>² = <strong>0.8386</strong></p>
        </div>
        <p class="note">
          其中：<em>x</em> 为成虫诱捕量（只/诱捕器），<em>y</em> 为预测的林间幼虫量。
        </p>
      </section>

      <section class="input-section">
        <h2><span class="section-num">2</span> 参数输入</h2>
        <div class="input-row">
          <label for="xInput">成虫诱捕量 x</label>
          <div class="input-with-unit">
            <input
              id="xInput"
              type="number"
              v-model.number="x"
              :min="minX"
              :max="maxX"
              step="1"
              @change="x = Math.round(Math.min(maxX, Math.max(minX, x)))"
            />
            <span class="unit">只 / 诱捕器</span>
          </div>
        </div>

        <input
          class="slider"
          type="range"
          v-model.number="x"
          :min="minX"
          :max="maxX"
          step="1"
        />

        <div class="range-labels">
          <span>{{ minX }}</span>
          <span>当前：{{ Math.round(x) }}</span>
          <span>{{ maxX }}</span>
        </div>
      </section>

      <div class="content-grid">
        <div class="left-col">
          <section class="result-section">
            <h2><span class="section-num">3</span> 预测结果</h2>
            <div class="result-card">
              <div class="result-row">
                <span class="result-value">{{ y.toFixed(2) }}</span>
                <span class="result-unit">只</span>
              </div>
              <div class="result-label">预测林间幼虫量</div>
            </div>
            <p class="hint">
              本结果基于统计回归模型，仅供监测分析与趋势判断参考。
            </p>
          </section>
        </div>

        <div class="right-col">
          <section class="chart-section">
            <h2><span class="section-num">4</span> 模型拟合曲线</h2>
            <p class="chart-desc">
              可拖动图中标记点或使用上方滑块调整 <em>x</em>，观察预测值随成虫诱捕量的变化趋势。
            </p>
            <div class="chart-container">
              <svg
                ref="svgRef"
                :viewBox="`0 0 ${chartWidth} ${chartHeight}`"
                class="chart-svg"
                @pointerdown="handlePointerDown"
                @pointermove="handlePointerMove"
                @pointerleave="stopDragging"
              >
                <defs>
                  <linearGradient id="bgGradient" x1="0" x2="0" y1="0" y2="1">
                    <stop offset="0%" stop-color="#fafbfc" />
                    <stop offset="100%" stop-color="#ffffff" />
                  </linearGradient>
                </defs>

                <rect
                  :x="padding"
                  :y="padding"
                  :width="chartWidth - padding * 2"
                  :height="chartHeight - padding * 2"
                  fill="url(#bgGradient)"
                  rx="8"
                />

                <g class="grid">
                  <line
                    v-for="i in 4"
                    :key="`h-${i}`"
                    :x1="padding"
                    :x2="chartWidth - padding"
                    :y1="padding + ((chartHeight - padding * 2) * i) / 4"
                    :y2="padding + ((chartHeight - padding * 2) * i) / 4"
                    stroke="#e5e7eb"
                    stroke-width="1"
                    stroke-dasharray="4 4"
                  />
                </g>

                <polyline
                  v-if="svgPoints.length"
                  :points="svgPoints.map((p) => `${p.x},${p.y}`).join(' ')"
                  fill="none"
                  stroke="#2563eb"
                  stroke-width="2.5"
                />

                <line
                  :x1="currentPoint.x"
                  :x2="currentPoint.x"
                  :y1="padding"
                  :y2="chartHeight - padding"
                  stroke="#60a5fa"
                  stroke-width="1.5"
                  stroke-dasharray="4 4"
                />

                <circle
                  :cx="currentPoint.x"
                  :cy="currentPoint.y"
                  r="7"
                  fill="#2563eb"
                  stroke="#ffffff"
                  stroke-width="2.5"
                />
              </svg>

              <div class="chart-legend">
                <div class="legend-item">
                  <span class="legend-dot"></span>
                  <span>二次拟合曲线</span>
                </div>
                <div class="legend-item">
                  <span class="legend-line"></span>
                  <span>当前 <em>x</em> 取值</span>
                </div>
              </div>
            </div>
          </section>
        </div>
      </div>
      </main>
    </div>
  </div>
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

// x 范围可根据你实际数据调整
const minX = 0
const maxX = 100

const x = ref(20)

const y = computed(() => {
  const v = x.value
  // y = 0.2633x² - 6.9109x + 51.8272
  return 0.2633 * v * v - 6.9109 * v + 51.8272
})

// 折线图尺寸与留白
const chartWidth = 640
const chartHeight = 260
const padding = 32

// 采样若干个点绘制模型曲线
const sampledPoints = computed(() => {
  const steps = 40
  const pts = []
  for (let i = 0; i <= steps; i++) {
    const xv = minX + ((maxX - minX) * i) / steps
    const yv = 0.2633 * xv * xv - 6.9109 * xv + 51.8272
    pts.push({ x: xv, y: yv })
  }
  return pts
})

const minY = computed(() => Math.min(...sampledPoints.value.map((p) => p.y)))
const maxY = computed(() => Math.max(...sampledPoints.value.map((p) => p.y)))

// 将真实坐标转换为 SVG 坐标
const svgPoints = computed(() => {
  const innerW = chartWidth - padding * 2
  const innerH = chartHeight - padding * 2
  const ySpan = maxY.value - minY.value || 1

  return sampledPoints.value.map((p) => {
    const nx = (p.x - minX) / (maxX - minX || 1)
    const ny = (p.y - minY.value) / ySpan
    return {
      x: padding + nx * innerW,
      y: chartHeight - padding - ny * innerH,
    }
  })
})

// 当前选中点在图中的位置
const currentPoint = computed(() => {
  const innerW = chartWidth - padding * 2
  const innerH = chartHeight - padding * 2
  const ySpan = maxY.value - minY.value || 1

  const nx = (x.value - minX) / (maxX - minX || 1)
  const ny = (y.value - minY.value) / ySpan

  return {
    x: padding + nx * innerW,
    y: chartHeight - padding - ny * innerH,
  }
})

// 图上的拖动交互
const svgRef = ref(null)
const isDragging = ref(false)

const clamp = (val, min, max) => {
  if (val < min) return min
  if (val > max) return max
  return val
}

const updateXFromEvent = (evt) => {
  const el = svgRef.value
  if (!el) return
  const rect = el.getBoundingClientRect()
  const innerW = chartWidth - padding * 2
  const relativeX = evt.clientX - rect.left - padding
  const ratio = clamp(relativeX / innerW, 0, 1)
  const newX = minX + ratio * (maxX - minX)
  x.value = Math.round(newX)
}

const handlePointerDown = (evt) => {
  isDragging.value = true
  updateXFromEvent(evt)
}

const handlePointerMove = (evt) => {
  if (!isDragging.value) return
  updateXFromEvent(evt)
}

const stopDragging = () => {
  isDragging.value = false
}

onMounted(() => {
  window.addEventListener('pointerup', stopDragging)
})

onBeforeUnmount(() => {
  window.removeEventListener('pointerup', stopDragging)
})
</script>

<style scoped>
/* 商业化扁平风格 */
.page {
  height: 100vh;
  width: 100%;
  margin: 0;
  padding: 24px;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  background: #f1f5f9;
  background-attachment: fixed;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC', 'Hiragino Sans GB', sans-serif;
}

.card {
  width: 100%;
  max-width: 960px;
  max-height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05), 0 4px 12px rgba(0, 0, 0, 0.04);
  border: none;
  color: #1f2937;
}

.card-header {
  flex-shrink: 0;
  padding: 28px 32px 20px;
  border-bottom: 1px solid #e5e7eb;
  background: #ffffff;
  position: relative;
}

.card-header-sticky {
  position: sticky;
  top: 0;
  z-index: 10;
}

.card-body {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  padding: 24px 32px 32px;
  padding-top: 20px;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.card-body::-webkit-scrollbar {
  display: none;
}

.header-badge {
  display: inline-block;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.05em;
  color: #2563eb;
  background: #eff6ff;
  padding: 5px 12px;
  border-radius: 6px;
  margin-bottom: 12px;
}

.card-header h1 {
  margin: 0 0 4px;
  font-size: 20px;
  font-weight: 600;
  letter-spacing: -0.01em;
  color: #111827;
  line-height: 1.35;
}

.subtitle {
  margin: 0;
  font-size: 13px;
  color: #6b7280;
}

section {
  margin-bottom: 28px;
}

section:last-of-type {
  margin-bottom: 0;
}

h2 {
  margin: 0 0 14px;
  font-size: 14px;
  font-weight: 600;
  color: #374151;
  letter-spacing: -0.01em;
}

.section-num {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  margin-right: 8px;
  font-size: 11px;
  font-weight: 600;
  background: #2563eb;
  color: #fff;
  border-radius: 6px;
}

/* 模型公式 */
.model-section .formula-box {
  padding: 18px 22px;
  margin: 8px 0 12px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
}

.model-section .formula {
  margin: 0 0 10px;
  font-size: 16px;
  font-weight: 500;
  color: #1f2937;
}

.model-section .formula em,
.model-section .note em {
  font-style: normal;
  font-weight: 600;
  color: #374151;
}

.model-section .coef {
  color: #2563eb;
  font-weight: 600;
}

.model-section .r2 {
  margin: 0;
  font-size: 13px;
  color: #6b7280;
}

.model-section .note {
  margin: 0;
  font-size: 12px;
  color: #9ca3af;
}

/* 输入区域 */
.input-section .input-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 14px;
}

.input-section label {
  font-size: 14px;
  color: #374151;
  white-space: nowrap;
  font-weight: 500;
}

.input-with-unit {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
  max-width: 160px;
}

.input-with-unit input {
  flex: 1;
  padding: 10px 14px;
  border-radius: 8px;
  border: 1px solid #e5e7eb;
  font-size: 14px;
  font-weight: 500;
  outline: none;
  transition: border-color 0.15s ease, box-shadow 0.15s ease;
  background: #ffffff;
}

.input-with-unit input:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.12);
}

.unit {
  font-size: 12px;
  color: #9ca3af;
}

/* 滑块 */
.slider {
  width: 100%;
  margin: 12px 0 8px;
  -webkit-appearance: none;
  appearance: none;
  height: 6px;
  border-radius: 6px;
  background: #e5e7eb;
  outline: none;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 18px;
  height: 18px;
  background: #2563eb;
  border-radius: 50%;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12);
  cursor: pointer;
}

.slider::-moz-range-thumb {
  width: 18px;
  height: 18px;
  background: #2563eb;
  border-radius: 50%;
  border: none;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12);
  cursor: pointer;
}

.range-labels {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #9ca3af;
}

/* 结果展示 */
.result-section {
  margin-bottom: 0;
}

.result-card {
  margin-top: 8px;
  padding: 24px 28px;
  border-radius: 10px;
  background: #eff6ff;
  border: 1px solid #bfdbfe;
  display: inline-flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
}

.result-row {
  display: flex;
  align-items: baseline;
  gap: 6px;
}

.result-value {
  font-size: 30px;
  font-weight: 700;
  color: #1d4ed8;
  letter-spacing: -0.02em;
}

.result-unit {
  font-size: 14px;
  color: #3b82f6;
  font-weight: 500;
}

.result-label {
  font-size: 12px;
  color: #6b7280;
  margin-top: 2px;
}

.hint {
  margin-top: 10px;
  font-size: 12px;
  color: #9ca3af;
}

/* 折线图区域 */
.chart-section {
  margin-bottom: 0;
}

.chart-desc {
  margin: 0 0 14px;
  font-size: 12px;
  color: #6b7280;
  line-height: 1.5;
}

.chart-container {
  border-radius: 10px;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  padding: 18px 20px 20px;
}

.chart-svg {
  width: 100%;
  height: auto;
  display: block;
}

.chart-legend {
  margin-top: 14px;
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  font-size: 12px;
  color: #6b7280;
}

.legend-item {
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.legend-dot {
  width: 10px;
  height: 10px;
  border-radius: 4px;
  background: #2563eb;
}

.legend-line {
  width: 20px;
  height: 0;
  border-top: 2px dashed #60a5fa;
}

/* 自适应布局 */
.content-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 24px;
}

.content-grid section {
  margin-bottom: 0;
}

.left-col,
.right-col {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

@media (min-width: 900px) {
  .content-grid {
    grid-template-columns: 1fr 1.4fr;
    align-items: start;
  }

  .input-section {
    margin-bottom: 24px;
  }
}

/* 响应式 */
@media (max-width: 640px) {
  .page {
    padding: 16px 12px;
  }

  .card-header {
    padding: 24px 20px 16px;
  }

  .card-body {
    padding: 20px 20px 24px;
  }

  .card-header h1 {
    font-size: 18px;
  }

  .model-section .formula-box {
    padding: 14px 18px;
  }

  .model-section .formula {
    font-size: 15px;
  }

  .input-section .input-row {
    flex-direction: column;
    align-items: flex-start;
  }

  .input-with-unit {
    max-width: 100%;
  }

  .result-card {
    width: 100%;
  }

  .result-value {
    font-size: 24px;
  }
}

/* 横屏手机/小高度屏幕 */
@media (max-height: 620px) {
  .page {
    align-items: flex-start;
  }

  .card {
    margin-top: 8px;
    max-height: calc(100vh - 16px);
  }
}

/* 横屏设备优先铺满宽度（网页/平板/横屏手机） */
@media (orientation: landscape) and (max-width: 1200px) {
  .page {
    padding: 16px 18px;
    align-items: flex-start;
  }

  .card {
    max-width: 100%;
  }
}
</style>
