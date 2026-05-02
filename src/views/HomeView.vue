<template>
  <div class="home-page">
    <!-- 极光背景 -->
    <div class="aurora aurora--green" />
    <div class="aurora aurora--pink" />
    <div class="grid-overlay" />

    <!-- 顶部导航 -->
    <nav class="navbar">
      <div class="nav-logo" @click="router.push('/')">
        <div class="logo-orb">
          <span class="logo-flame">🔥</span>
          <span class="logo-ring" />
        </div>
        <span class="logo-name">FitGenie</span>
      </div>
      <div class="nav-actions">
        <router-link to="/history" class="nav-link">历史记录</router-link>
        <el-button size="small" @click="handleLogout">退出</el-button>
      </div>
    </nav>

    <div class="content">
      <!-- 欢迎区 -->
      <div class="hero">
        <div class="hero-left">
          <p class="greeting">
            <span class="dot-pulse" />
            {{ greeting }}
          </p>
          <h1 class="hero-name">
            <span class="gradient-text">{{ profile?.name || '...' }}</span>
          </h1>
          <p class="hero-sub">
            今天是你健身旅程的
            <span class="highlight">第 {{ dayCount }} 天</span>
          </p>
        </div>
        <div class="hero-right">
          <div class="streak-card glow-border">
            <div class="streak-flame-bg">🔥</div>
            <div class="streak-num">{{ streak }}</div>
            <div class="streak-label">连续打卡</div>
            <div class="streak-unit">DAYS</div>
          </div>
        </div>
      </div>

      <!-- 数据概览 -->
      <div class="stats-row">
        <div
          v-for="(stat, i) in statCards"
          :key="stat.label"
          class="stat-card card"
          :style="{ animationDelay: `${0.1 + i * 0.08}s` }"
        >
          <div class="stat-glow" :style="{ background: `radial-gradient(circle at 50% 0%, ${stat.color}55, transparent 70%)` }" />
          <div class="stat-icon">{{ stat.icon }}</div>
          <div class="stat-value">{{ stat.value }}</div>
          <div class="stat-label">{{ stat.label }}</div>
          <div class="stat-bar">
            <div
              class="stat-bar-fill"
              :style="{ width: stat.progress + '%', background: `linear-gradient(90deg, ${stat.color}, var(--pink))` }"
            />
          </div>
        </div>
      </div>

      <!-- 快速操作 -->
      <div class="actions-section">
        <h2 class="section-title">⚡ 今天准备好了吗？</h2>
        <div class="action-cards">
          <div class="action-card action-card--primary glow-border" @click="router.push('/checkin')">
            <div class="action-shine" />
            <div class="action-icon">⚡</div>
            <div class="action-content">
              <div class="action-title">START TODAY'S BURN</div>
              <div class="action-desc">输入今日数据，AI 即刻生成减脂方案</div>
            </div>
            <div class="action-arrow">→</div>
          </div>

          <div class="action-card card" @click="router.push('/history')">
            <div class="action-icon">📊</div>
            <div class="action-content">
              <div class="action-title">查看历史</div>
              <div class="action-desc">趋势 · 回顾</div>
            </div>
            <div class="action-arrow">→</div>
          </div>
        </div>
      </div>

      <!-- 最新方案预览 -->
      <div v-if="lastCheckin" class="last-plan">
        <h2 class="section-title">📋 上次的方案</h2>
        <div class="plan-cards">
          <div class="plan-card card">
            <div class="plan-tag">💪 WORKOUT</div>
            <p class="plan-text">{{ lastCheckin.workout_plan }}</p>
          </div>
          <div class="plan-card card">
            <div class="plan-tag">🥗 NUTRITION</div>
            <p class="plan-text">{{ lastCheckin.diet_plan }}</p>
          </div>
        </div>
        <div class="motivation card">
          <div class="motivation-bar" />
          <span class="motivation-icon">✨</span>
          <p class="motivation-text">{{ lastCheckin.motivation_message }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { profileApi, dailyApi } from '@/api'
import { useAuthStore } from '@/stores/auth'
import type { UserProfile, DailyLogItem } from '@/api/types'

const router = useRouter()
const authStore = useAuthStore()

const profile = ref<UserProfile | null>(null)
const history = ref<DailyLogItem[]>([])
const lastCheckin = ref<any>(null)
const streak = ref(0)
const dayCount = ref(1)

const greeting = computed(() => {
  const h = new Date().getHours()
  if (h < 6) return '夜深了'
  if (h < 11) return '早上好'
  if (h < 14) return '午安'
  if (h < 18) return '下午好'
  return '晚上好'
})

const statCards = computed(() => {
  const latest = history.value[0]
  return [
    {
      icon: '⚖️',
      label: 'CURRENT WEIGHT',
      value: latest?.weight_kg ? `${latest.weight_kg} kg` : '--',
      progress: latest?.weight_kg ? Math.min(100, (latest.weight_kg / 100) * 100) : 0,
      color: '#10dba1',
    },
    {
      icon: '👟',
      label: 'STEPS YESTERDAY',
      value: latest?.steps ? `${latest.steps.toLocaleString()}` : '--',
      progress: latest?.steps ? Math.min(100, (latest.steps / 10000) * 100) : 0,
      color: '#ff4d8d',
    },
    {
      icon: '🔥',
      label: 'CALORIES INTAKE',
      value: latest?.calories_intake ? `${latest.calories_intake}` : '--',
      progress: latest?.calories_intake ? Math.min(100, (latest.calories_intake / 2500) * 100) : 0,
      color: '#c084fc',
    },
    {
      icon: '🏋️',
      label: 'WORKOUT',
      value: latest?.workout_done ? 'DONE' : (latest ? 'SKIP' : '--'),
      progress: latest?.workout_done ? 100 : 0,
      color: '#00ffaa',
    },
  ]
})

function handleLogout() {
  authStore.logout()
  router.push('/login')
}

onMounted(async () => {
  try {
    const [profileRes, historyRes] = await Promise.all([
      profileApi.get(),
      dailyApi.history(30),
    ])
    profile.value = profileRes.data
    history.value = historyRes.data

    let s = 0
    for (const log of historyRes.data) {
      if (log.workout_done) s++
      else break
    }
    streak.value = s
    dayCount.value = historyRes.data.length + 1

    const saved = localStorage.getItem('lastCheckin')
    if (saved) lastCheckin.value = JSON.parse(saved)
  } catch (e) {
    ElMessage({ message: '加载数据失败', type: 'error' })
  }
})
</script>

<style scoped>
.home-page {
  min-height: 100vh;
  position: relative;
  overflow-x: hidden;
  z-index: 0;
}

/* 极光 */
.aurora {
  position: fixed;
  border-radius: 50%;
  filter: blur(100px);
  pointer-events: none;
  opacity: 0.4;
  mix-blend-mode: screen;
}
.aurora--green {
  width: 720px; height: 720px;
  background: radial-gradient(circle, #10dba1, transparent 60%);
  top: -260px; right: -200px;
  animation: aurora 24s linear infinite;
}
.aurora--pink {
  width: 600px; height: 600px;
  background: radial-gradient(circle, #ff4d8d, transparent 60%);
  bottom: -200px; left: -200px;
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

/* 导航 */
.navbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 40px;
  position: sticky;
  top: 0;
  z-index: 100;
  background: rgba(5, 8, 19, 0.7);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border-bottom: 1px solid var(--border);
}
.nav-logo {
  display: flex;
  align-items: center;
  gap: 12px;
  cursor: pointer;
}
.logo-orb {
  width: 36px; height: 36px;
  border-radius: 50%;
  background: var(--gradient-primary);
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  box-shadow: var(--glow-mix);
  animation: pulse-glow 3s ease-in-out infinite;
}
.logo-flame {
  font-size: 16px;
  animation: flame-flicker 2s ease-in-out infinite;
}
.logo-ring {
  position: absolute;
  inset: -3px;
  border-radius: 50%;
  border: 2px solid var(--green-bright);
  opacity: 0;
  animation: ring-pulse 2.5s ease-out infinite;
}
.logo-name {
  font-family: var(--font-display);
  font-size: 20px;
  font-weight: 700;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradient-shift 5s ease-in-out infinite;
}
.nav-actions {
  display: flex;
  align-items: center;
  gap: 24px;
}
.nav-link {
  color: var(--text-secondary);
  text-decoration: none;
  font-size: 13px;
  font-weight: 500;
  letter-spacing: 0.04em;
  transition: var(--transition);
  position: relative;
}
.nav-link::after {
  content: '';
  position: absolute;
  bottom: -4px; left: 0; right: 0;
  height: 1.5px;
  background: var(--gradient-primary);
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 0.3s ease;
}
.nav-link:hover { color: var(--green-bright); }
.nav-link:hover::after { transform: scaleX(1); }

.content {
  max-width: 960px;
  margin: 0 auto;
  padding: 48px 24px 96px;
  position: relative;
  z-index: 2;
}

/* 欢迎区 */
.hero {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 56px;
  animation: fade-up 0.7s var(--ease-out);
}
.greeting {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--text-muted);
  font-family: var(--font-mono);
  font-size: 12px;
  letter-spacing: 0.16em;
  text-transform: uppercase;
  margin-bottom: 12px;
}
.dot-pulse {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: var(--green-bright);
  box-shadow: 0 0 10px var(--green-bright);
  animation: pulse-soft 1.6s ease-in-out infinite;
}
.hero-name {
  font-family: var(--font-display);
  font-size: 64px;
  font-weight: 800;
  margin-bottom: 12px;
  letter-spacing: -0.03em;
  line-height: 1;
}
.hero-sub {
  color: var(--text-secondary);
  font-size: 15px;
}
.highlight {
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 700;
  animation: gradient-shift 4s ease-in-out infinite;
}

/* 连续打卡卡片 */
.streak-card {
  width: 160px;
  height: 160px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
  cursor: default;
  background: var(--bg-glass);
  backdrop-filter: blur(20px);
  border-radius: var(--radius);
  padding: 20px;
}
.streak-flame-bg {
  position: absolute;
  font-size: 120px;
  opacity: 0.07;
  animation: flame-flicker 3s ease-in-out infinite;
}
.streak-num {
  font-family: var(--font-display);
  font-size: 56px;
  font-weight: 800;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  line-height: 1;
  animation: gradient-shift 4s ease-in-out infinite;
  position: relative;
  z-index: 1;
}
.streak-label {
  font-size: 11px;
  color: var(--text-secondary);
  letter-spacing: 0.1em;
  margin-top: 4px;
  position: relative;
  z-index: 1;
}
.streak-unit {
  font-family: var(--font-mono);
  font-size: 10px;
  color: var(--text-muted);
  letter-spacing: 0.2em;
  font-weight: 600;
  position: relative;
  z-index: 1;
}

/* 数据概览 */
.stats-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin-bottom: 56px;
}
.stat-card {
  padding: 22px;
  animation: fade-up 0.7s var(--ease-out) both;
  position: relative;
  overflow: hidden;
}
.stat-glow {
  position: absolute;
  inset: 0;
  opacity: 0;
  transition: opacity 0.4s ease;
  pointer-events: none;
}
.stat-card:hover .stat-glow { opacity: 0.5; }
.stat-icon {
  font-size: 22px;
  margin-bottom: 12px;
  filter: drop-shadow(0 0 8px rgba(16,219,161,0.4));
}
.stat-value {
  font-family: var(--font-display);
  font-size: 24px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 4px;
  letter-spacing: -0.01em;
}
.stat-label {
  font-family: var(--font-mono);
  font-size: 10px;
  color: var(--text-muted);
  margin-bottom: 14px;
  letter-spacing: 0.12em;
  font-weight: 600;
}
.stat-bar {
  height: 4px;
  background: rgba(255,255,255,0.05);
  border-radius: 2px;
  overflow: hidden;
}
.stat-bar-fill {
  height: 100%;
  border-radius: 2px;
  transition: width 1.2s var(--ease-out);
  box-shadow: 0 0 8px currentColor;
}

/* 快速操作 */
.section-title {
  font-family: var(--font-display);
  font-size: 24px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 20px;
  letter-spacing: -0.01em;
}
.action-cards {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 16px;
  margin-bottom: 56px;
}
.action-card {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 28px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  border-radius: var(--radius);
  transition: var(--transition);
}
.action-card.card:hover { transform: translateY(-4px); }
.action-card--primary {
  background: linear-gradient(135deg, rgba(16,219,161,0.18), rgba(255,77,141,0.18)) !important;
  border: none !important;
  box-shadow: var(--glow-mix);
}
.action-card--primary:hover {
  transform: translateY(-4px) !important;
  box-shadow: 0 0 60px rgba(16,219,161,0.5), 0 0 120px rgba(255,77,141,0.3) !important;
}
.action-shine {
  position: absolute;
  top: 0; left: -100%;
  width: 50%; height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.15), transparent);
  animation: shimmer 3s ease-in-out infinite;
}
.action-icon {
  font-size: 36px;
  flex-shrink: 0;
  filter: drop-shadow(0 0 12px rgba(0,255,170,0.5));
}
.action-card--primary .action-icon {
  animation: flame-flicker 2s ease-in-out infinite;
}
.action-content { position: relative; z-index: 1; }
.action-title {
  font-family: var(--font-display);
  font-size: 16px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 4px;
  letter-spacing: 0.02em;
}
.action-card--primary .action-title { color: var(--text-on-grad); }
.action-desc {
  font-size: 13px;
  color: var(--text-secondary);
}
.action-arrow {
  margin-left: auto;
  font-size: 22px;
  color: var(--text-muted);
  transition: var(--transition);
  font-weight: 300;
}
.action-card--primary .action-arrow { color: white; }
.action-card:hover .action-arrow { transform: translateX(6px); color: var(--green-bright); }

/* 上次方案 */
.plan-cards {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-bottom: 16px;
}
.plan-card { padding: 24px; }
.plan-tag {
  font-family: var(--font-mono);
  font-size: 11px;
  font-weight: 700;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  letter-spacing: 0.16em;
  margin-bottom: 14px;
  animation: gradient-shift 4s ease-in-out infinite;
}
.plan-text {
  font-size: 13px;
  color: var(--text-secondary);
  line-height: 1.8;
  white-space: pre-line;
  max-height: 180px;
  overflow-y: auto;
}
.motivation {
  display: flex;
  align-items: flex-start;
  gap: 16px;
  padding: 24px;
  background: linear-gradient(135deg, rgba(16,219,161,0.08), rgba(255,77,141,0.08)) !important;
  border-color: var(--border-bright) !important;
  position: relative;
  overflow: hidden;
}
.motivation-bar {
  position: absolute;
  left: 0; top: 0; bottom: 0;
  width: 3px;
  background: var(--gradient-primary);
  box-shadow: var(--glow-mix);
}
.motivation-icon {
  font-size: 24px;
  flex-shrink: 0;
  animation: pulse-soft 2s ease-in-out infinite;
}
.motivation-text {
  font-size: 14px;
  color: var(--text-secondary);
  line-height: 1.8;
  font-style: italic;
}

@media (max-width: 768px) {
  .navbar { padding: 16px 20px; }
  .content { padding: 32px 16px 80px; }
  .hero { flex-direction: column; align-items: flex-start; gap: 32px; }
  .hero-name { font-size: 48px; }
  .stats-row { grid-template-columns: repeat(2, 1fr); }
  .action-cards { grid-template-columns: 1fr; }
  .plan-cards { grid-template-columns: 1fr; }
}
</style>
