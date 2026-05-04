<template>
  <div class="history-page">
    <div class="aurora aurora--green" />
    <div class="aurora aurora--pink" />
    <div class="grid-overlay" />

    <nav class="navbar">
      <div class="nav-left" @click="router.push('/')">
        <span class="back-arrow">←</span>
        <div class="logo-orb">
          <span class="logo-flame">🔥</span>
        </div>
        <span class="logo-name">FitGenie</span>
      </div>
      <div class="nav-title">
        <span class="dot-pulse" />
        TIMELINE · HISTORY
      </div>
    </nav>

    <div class="content">
      <div class="page-header">
        <div class="page-tag">
          <span class="tag-dot" />
          YOUR JOURNEY
        </div>
        <h1 class="page-title">
          每一天的<span class="gradient-text">坚持</span>
        </h1>
        <p class="page-sub">数据不会撒谎，你的进步都在这里</p>
      </div>

      <!-- 加载中 -->
      <div v-if="loading" class="loading-state">
        <div class="loading-dots">
          <span /><span /><span />
        </div>
        <p>加载数据中...</p>
      </div>

      <!-- 空状态 -->
      <div v-else-if="!logs.length" class="empty-state card">
        <div class="empty-icon">🌱</div>
        <h3 class="empty-title">还没有记录</h3>
        <p class="empty-desc">完成第一次打卡，开始你的减脂旅程</p>
        <el-button type="primary" size="large" @click="router.push('/checkin')">
          ⚡ 立即打卡
        </el-button>
      </div>

      <template v-else>
        <!-- 统计概览 -->
        <div class="overview-grid">
          <div
            v-for="(stat, i) in overviewStats"
            :key="stat.label"
            class="overview-card card"
            :style="{ animationDelay: `${i * 0.08}s` }"
          >
            <div class="overview-icon">{{ stat.icon }}</div>
            <div class="overview-value gradient-text">{{ stat.value }}</div>
            <div class="overview-label">{{ stat.label }}</div>
          </div>
        </div>

        <!-- 体重趋势图 -->
        <div class="chart-section card">
          <div class="chart-header">
            <div class="chart-title">📈 WEIGHT TREND</div>
            <div class="chart-legend">
              <span class="legend-dot" />
              <span class="legend-label">体重 (kg)</span>
            </div>
          </div>
          <div class="chart-wrap">
            <svg
              class="weight-chart"
              viewBox="0 0 600 160"
              preserveAspectRatio="none"
            >
              <line
                v-for="i in 4"
                :key="i"
                x1="0" :y1="i * 32"
                x2="600" :y2="i * 32"
                stroke="rgba(255,255,255,0.04)"
                stroke-width="1"
              />
              <path
                v-if="chartPath"
                :d="chartArea"
                fill="url(#areaGrad)"
              />
              <path
                v-if="chartPath"
                :d="chartPath"
                fill="none"
                stroke="url(#lineGrad)"
                stroke-width="2.5"
                stroke-linecap="round"
                stroke-linejoin="round"
                class="chart-line"
              />
              <circle
                v-for="(pt, i) in chartPoints"
                :key="i"
                :cx="pt.x" :cy="pt.y"
                r="4.5"
                fill="#050813"
                stroke="url(#lineGrad)"
                stroke-width="2"
                class="chart-dot"
              >
                <title>{{ chartLabels[i] }}: {{ pt.weight }}kg</title>
              </circle>
              <defs>
                <linearGradient id="lineGrad" x1="0" y1="0" x2="1" y2="0">
                  <stop offset="0%" stop-color="#10dba1" />
                  <stop offset="100%" stop-color="#ff4d8d" />
                </linearGradient>
                <linearGradient id="areaGrad" x1="0" y1="0" x2="0" y2="1">
                  <stop offset="0%" stop-color="#10dba1" stop-opacity="0.25" />
                  <stop offset="100%" stop-color="#ff4d8d" stop-opacity="0" />
                </linearGradient>
              </defs>
            </svg>
            <div class="chart-xlabels">
              <span
                v-for="(label, i) in chartLabels"
                :key="i"
                class="chart-xlabel"
              >{{ label }}</span>
            </div>
          </div>
        </div>

        <!-- 日志列表 -->
        <div class="logs-section">
          <div class="logs-title">📋 CHECK-IN LOG</div>
          <div class="logs-list">
            <div
              v-for="(log, i) in logs"
              :key="log.date"
              class="log-item card"
              :style="{ animationDelay: `${i * 0.04}s` }"
            >
              <div class="log-date">
                <div class="log-day">{{ formatDay(log.date) }}</div>
                <div class="log-month">{{ formatMonth(log.date) }}</div>
              </div>
              <div class="log-divider" />
              <div class="log-stats">
                <div class="log-stat">
                  <span class="log-stat-icon">⚖️</span>
                  <span class="log-stat-val">{{ log.weight_kg ?? '--' }} kg</span>
                </div>
                <div class="log-stat">
                  <span class="log-stat-icon">👟</span>
                  <span class="log-stat-val">{{ log.steps?.toLocaleString() ?? '--' }}</span>
                </div>
                <div class="log-stat">
                  <span class="log-stat-icon">🍽️</span>
                  <span class="log-stat-val">{{ log.calories_intake ?? '--' }} kcal</span>
                </div>
              </div>
              <div class="log-badges">
                <span
                  class="log-badge"
                  :class="log.workout_done ? 'badge--done' : 'badge--skip'"
                >
                  {{ log.workout_done ? '✓ TRAINED' : '✗ REST' }}
                </span>
                <span class="log-badge badge--mood">
                  {{ moodEmoji[log.mood ?? 'neutral'] }}
                  {{ moodLabel[log.mood ?? 'neutral'] }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { dailyApi } from '@/api'
import type { DailyLogItem } from '@/api/types'

const router = useRouter()
const loading = ref(true)
const logs = ref<DailyLogItem[]>([])

const moodEmoji: Record<string, string> = {
  good: '😄', neutral: '😐', tired: '😴',
}
const moodLabel: Record<string, string> = {
  good: 'GOOD', neutral: 'OK', tired: 'TIRED',
}

const overviewStats = computed(() => {
  const validWeights = logs.value
    .map(l => l.weight_kg)
    .filter((w): w is number => w !== null)

  const workoutDays = logs.value.filter(l => l.workout_done).length
  const totalDays = logs.value.length

  const weightChange = validWeights.length >= 2
    ? ((validWeights[validWeights.length - 1]! - validWeights[0]!)).toFixed(1)
    : '--'

  const avgSteps = logs.value.length
    ? Math.round(logs.value.reduce((s, l) => s + (l.steps ?? 0), 0) / logs.value.length)
    : 0

  return [
    { icon: '📅', label: 'TOTAL DAYS', value: `${totalDays}` },
    { icon: '🏋️', label: 'WORKOUTS', value: `${workoutDays}` },
    {
      icon: '📉',
      label: 'WEIGHT Δ',
      value: weightChange === '--' ? '--' : `${Number(weightChange) > 0 ? '+' : ''}${weightChange} kg`,
    },
    { icon: '👟', label: 'AVG STEPS', value: avgSteps ? avgSteps.toLocaleString() : '--' },
  ]
})

const chartPoints = computed(() => {
  const weightLogs = [...logs.value]
    .reverse()
    .filter(l => l.weight_kg !== null)
    .slice(-14)

  if (weightLogs.length < 2) return []

  const weights = weightLogs.map(l => l.weight_kg as number)
  const minW = Math.min(...weights) - 1
  const maxW = Math.max(...weights) + 1
  const range = maxW - minW || 1

  return weightLogs.map((log, i) => ({
    x: (i / (weightLogs.length - 1)) * 580 + 10,
    y: 140 - ((log.weight_kg! - minW) / range) * 120,
    weight: log.weight_kg,
  }))
})

const chartLabels = computed(() =>
  [...logs.value]
    .reverse()
    .filter(l => l.weight_kg !== null)
    .slice(-14)
    .map(l => formatDay(l.date))
)

const chartPath = computed(() => {
  const pts = chartPoints.value
  if (pts.length < 2) return ''
  return pts.map((p, i) => `${i === 0 ? 'M' : 'L'} ${p.x} ${p.y}`).join(' ')
})

const chartArea = computed(() => {
  const pts = chartPoints.value
  if (pts.length < 2) return ''
  const line = pts.map((p, i) => `${i === 0 ? 'M' : 'L'} ${p.x} ${p.y}`).join(' ')
  const last = pts[pts.length - 1]!
  const first = pts[0]!
  return `${line} L ${last.x} 160 L ${first.x} 160 Z`
})

function formatDay(dateStr: string) {
  return new Date(dateStr).getDate() + ''
}
function formatMonth(dateStr: string) {
  return (new Date(dateStr).getMonth() + 1) + '月'
}

onMounted(async () => {
  try {
    const res = await dailyApi.history(30)
    logs.value = res.data
  } catch {
    ElMessage({ message: '加载失败', type: 'error' })
  } finally {
    loading.value = false
  }
})
</script>

<style scoped>
.history-page {
  min-height: 100vh;
  position: relative;
  overflow-x: hidden;
  z-index: 0;
}

.aurora {
  position: fixed;
  border-radius: 50%;
  filter: blur(100px);
  pointer-events: none;
  opacity: 0.4;
  mix-blend-mode: screen;
}
.aurora--green {
  width: 700px; height: 700px;
  background: radial-gradient(circle, #10dba1, transparent 60%);
  top: -200px; right: -180px;
  animation: aurora 22s linear infinite;
}
.aurora--pink {
  width: 600px; height: 600px;
  background: radial-gradient(circle, #ff4d8d, transparent 60%);
  bottom: -200px; left: -180px;
  animation: aurora 18s linear infinite reverse;
}
.grid-overlay {
  position: fixed; inset: 0;
  background-image:
    linear-gradient(rgba(255,255,255,0.025) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,0.025) 1px, transparent 1px);
  background-size: 50px 50px;
  pointer-events: none;
  mask-image: radial-gradient(circle at center, black 30%, transparent 90%);
}

.navbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 40px;
  position: sticky; top: 0; z-index: 100;
  background: rgba(5, 8, 19, 0.7);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border-bottom: 1px solid var(--border);
}
.nav-left {
  display: flex;
  align-items: center;
  gap: 12px;
  cursor: pointer;
  color: var(--text-secondary);
  font-size: 14px;
  transition: var(--transition);
}
.nav-left:hover { color: var(--green-bright); }
.back-arrow { font-size: 20px; }
.logo-orb {
  width: 32px; height: 32px;
  border-radius: 50%;
  background: var(--gradient-primary);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: var(--glow-mix);
  animation: pulse-glow 3s ease-in-out infinite;
}
.logo-flame {
  font-size: 14px;
  animation: flame-flicker 2s ease-in-out infinite;
}
.logo-name {
  font-family: var(--font-display);
  font-size: 18px;
  font-weight: 700;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradient-shift 5s ease-in-out infinite;
}
.nav-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-family: var(--font-mono);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.18em;
  color: var(--text-secondary);
}
.dot-pulse {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: var(--green-bright);
  box-shadow: 0 0 10px var(--green-bright);
  animation: pulse-soft 1.6s ease-in-out infinite;
}

.content {
  max-width: 880px;
  margin: 0 auto;
  padding: 48px 24px 96px;
  position: relative;
  z-index: 2;
}

.page-header { margin-bottom: 40px; animation: fade-up 0.6s var(--ease-out); }
.page-tag {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-family: var(--font-mono);
  font-size: 11px;
  letter-spacing: 0.2em;
  color: var(--green);
  margin-bottom: 16px;
  padding: 4px 12px;
  border: 1px solid var(--border-bright);
  border-radius: var(--radius-pill);
  background: rgba(16,219,161,0.08);
}
.tag-dot {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: var(--green-bright);
  box-shadow: 0 0 8px var(--green-bright);
  animation: pulse-soft 1.4s ease-in-out infinite;
}
.page-title {
  font-family: var(--font-display);
  font-size: 48px;
  font-weight: 800;
  color: var(--text-primary);
  margin-bottom: 10px;
  letter-spacing: -0.02em;
  line-height: 1.05;
}
.page-sub { color: var(--text-secondary); font-size: 15px; }

/* 加载 */
.loading-state {
  text-align: center;
  padding: 100px 0;
  color: var(--text-muted);
}
.loading-dots {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}
.loading-dots span {
  width: 10px; height: 10px;
  border-radius: 50%;
  background: var(--gradient-primary);
  box-shadow: var(--glow-mix);
  animation: bounce 1.2s ease-in-out infinite;
}
.loading-dots span:nth-child(2) { animation-delay: 0.15s; }
.loading-dots span:nth-child(3) { animation-delay: 0.3s; }

/* 空状态 */
.empty-state {
  text-align: center;
  padding: 80px 40px;
}
.empty-icon {
  font-size: 64px;
  margin-bottom: 20px;
  filter: drop-shadow(0 0 20px rgba(16,219,161,0.4));
}
.empty-title {
  font-family: var(--font-display);
  font-size: 28px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 10px;
}
.empty-desc {
  color: var(--text-secondary);
  font-size: 14px;
  margin-bottom: 28px;
}

/* 概览 */
.overview-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin-bottom: 28px;
}
.overview-card {
  padding: 24px 20px;
  text-align: center;
  animation: fade-up 0.6s var(--ease-out) both;
}
.overview-icon {
  font-size: 26px;
  margin-bottom: 12px;
  filter: drop-shadow(0 0 8px rgba(16,219,161,0.4));
}
.overview-value {
  font-family: var(--font-display);
  font-size: 28px;
  font-weight: 800;
  margin-bottom: 6px;
  background-size: 200% auto;
  animation: gradient-shift 4s ease-in-out infinite;
  letter-spacing: -0.01em;
}
.overview-label {
  font-family: var(--font-mono);
  font-size: 10px;
  color: var(--text-muted);
  letter-spacing: 0.16em;
  font-weight: 600;
}

/* 图表 */
.chart-section {
  padding: 28px;
  margin-bottom: 28px;
  animation: fade-up 0.6s var(--ease-out) 0.3s both;
}
.chart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}
.chart-title {
  font-family: var(--font-mono);
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.18em;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradient-shift 4s ease-in-out infinite;
}
.chart-legend {
  display: flex;
  align-items: center;
  gap: 8px;
}
.legend-dot {
  width: 8px; height: 8px;
  border-radius: 50%;
  background: var(--gradient-primary);
  box-shadow: var(--glow-green);
}
.legend-label {
  font-family: var(--font-mono);
  font-size: 11px;
  color: var(--text-muted);
  letter-spacing: 0.08em;
}
.weight-chart {
  width: 100%;
  height: 160px;
  display: block;
  filter: drop-shadow(0 0 8px rgba(16,219,161,0.2));
}
.chart-line {
  stroke-dasharray: 1000;
  stroke-dashoffset: 1000;
  animation: drawLine 1.8s var(--ease-out) forwards;
}
.chart-dot {
  opacity: 0;
  animation: fadeIn 0.4s ease forwards;
}
.chart-xlabels {
  display: flex;
  justify-content: space-between;
  margin-top: 12px;
  padding: 0 10px;
}
.chart-xlabel {
  font-family: var(--font-mono);
  font-size: 10px;
  color: var(--text-muted);
}

/* 日志列表 */
.logs-title {
  font-family: var(--font-mono);
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.18em;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 16px;
  animation: gradient-shift 4s ease-in-out infinite;
}
.logs-list { display: flex; flex-direction: column; gap: 10px; }
.log-item {
  display: flex;
  align-items: center;
  gap: 24px;
  padding: 20px 24px;
  animation: fade-up 0.5s var(--ease-out) both;
}
.log-date {
  text-align: center;
  flex-shrink: 0;
  width: 50px;
}
.log-day {
  font-family: var(--font-display);
  font-size: 28px;
  font-weight: 800;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradient-shift 4s ease-in-out infinite;
  line-height: 1;
}
.log-month {
  font-family: var(--font-mono);
  font-size: 10px;
  color: var(--text-muted);
  margin-top: 4px;
  letter-spacing: 0.08em;
}
.log-divider {
  width: 1px;
  height: 44px;
  background: linear-gradient(to bottom, transparent, var(--border), transparent);
  flex-shrink: 0;
}
.log-stats {
  display: flex;
  gap: 24px;
  flex: 1;
  flex-wrap: wrap;
}
.log-stat {
  display: flex;
  align-items: center;
  gap: 8px;
}
.log-stat-icon { font-size: 14px; }
.log-stat-val {
  font-size: 13px;
  color: var(--text-secondary);
  font-weight: 600;
  font-family: var(--font-mono);
}
.log-badges {
  display: flex;
  gap: 8px;
  flex-shrink: 0;
}
.log-badge {
  padding: 5px 12px;
  border-radius: var(--radius-pill);
  font-family: var(--font-mono);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.08em;
  border: 1px solid;
}
.badge--done {
  background: rgba(16,219,161,0.1);
  color: var(--green-bright);
  border-color: var(--border-bright);
}
.badge--skip {
  background: rgba(255,77,141,0.08);
  color: var(--pink-bright);
  border-color: var(--border-pink);
}
.badge--mood {
  background: rgba(255,255,255,0.04);
  color: var(--text-secondary);
  border-color: var(--border);
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-12px); }
}
@keyframes drawLine {
  to { stroke-dashoffset: 0; }
}
@keyframes fadeIn {
  to { opacity: 1; }
}

@media (max-width: 768px) {
  .navbar { padding: 16px 20px; }
  .overview-grid { grid-template-columns: repeat(2, 1fr); }
  .log-item { flex-wrap: wrap; }
  .log-stats { flex-wrap: wrap; gap: 14px; }
  .page-title { font-size: 36px; }
}
</style>
