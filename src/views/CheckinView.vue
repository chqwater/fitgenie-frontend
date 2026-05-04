<template>
  <div class="checkin-page">
    <div class="aurora aurora--green" />
    <div class="aurora aurora--pink" />
    <div class="grid-overlay" />

    <!-- 导航 -->
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
        TODAY · CHECK-IN
      </div>
    </nav>

    <div class="content">
      <!-- 输入阶段 -->
      <transition name="fade" mode="out-in">
        <div v-if="!result" key="form" class="form-section">
          <div class="form-header">
            <div class="page-tag">
              <span class="tag-dot" />
              DAILY LOG · {{ today }}
            </div>
            <h1 class="page-title">
              记录<span class="gradient-text">今天</span>
            </h1>
            <p class="page-sub">告诉 AI 你的状态，它会为你重写今日方案</p>
          </div>

          <div class="input-grid">
            <!-- 体重 -->
            <div class="input-card card">
              <div class="input-label">
                <span class="input-icon">⚖️</span>
                WEIGHT
              </div>
              <div class="weight-input">
                <button class="adj-btn" @click="adjust('weight_kg', -0.1)">−</button>
                <div class="weight-display">
                  <span class="weight-num gradient-text">{{ form.weight_kg.toFixed(1) }}</span>
                  <span class="weight-unit">kg</span>
                </div>
                <button class="adj-btn" @click="adjust('weight_kg', 0.1)">+</button>
              </div>
            </div>

            <!-- 步数 -->
            <div class="input-card card">
              <div class="input-label">
                <span class="input-icon">👟</span>
                STEPS
              </div>
              <div class="steps-display">
                <el-slider
                  v-model="form.steps"
                  :min="0"
                  :max="20000"
                  :step="100"
                  :show-tooltip="true"
                  :format-tooltip="(v: number) => `${v.toLocaleString()} 步`"
                />
                <div class="steps-num gradient-text">{{ form.steps.toLocaleString() }} <span class="unit-sm">steps</span></div>
              </div>
            </div>

            <!-- 热量 -->
            <div class="input-card card">
              <div class="input-label">
                <span class="input-icon">🍽️</span>
                CALORIES INTAKE
              </div>
              <div class="weight-input">
                <button class="adj-btn" @click="adjust('calories_intake', -50)">−</button>
                <div class="weight-display">
                  <span class="weight-num gradient-text">{{ form.calories_intake }}</span>
                  <span class="weight-unit">kcal</span>
                </div>
                <button class="adj-btn" @click="adjust('calories_intake', 50)">+</button>
              </div>
            </div>

            <!-- 训练完成 -->
            <div class="input-card card">
              <div class="input-label">
                <span class="input-icon">🏋️</span>
                WORKOUT
              </div>
              <div class="workout-toggle">
                <div
                  class="toggle-option"
                  :class="{ active: form.workout_done }"
                  @click="form.workout_done = true"
                >
                  <span>✓</span> 完成
                </div>
                <div
                  class="toggle-option toggle-option--no"
                  :class="{ active: !form.workout_done }"
                  @click="form.workout_done = false"
                >
                  <span>✗</span> 休息
                </div>
              </div>
            </div>
          </div>

          <!-- 情绪选择 -->
          <div class="mood-section card">
            <div class="input-label">
              <span class="input-icon">💭</span>
              MOOD TODAY
            </div>
            <div class="mood-options">
              <div
                v-for="m in moods"
                :key="m.value"
                class="mood-card"
                :class="{ selected: form.mood === m.value }"
                @click="form.mood = m.value"
              >
                <div class="mood-emoji">{{ m.emoji }}</div>
                <div class="mood-label">{{ m.label }}</div>
              </div>
            </div>
          </div>

          <!-- 提交 -->
          <el-button
            class="submit-btn"
            type="primary"
            size="large"
            :loading="loading"
            @click="handleCheckin"
          >
            <span v-if="!loading">⚡ 生成今日方案</span>
            <span v-else>AI 正在重写你的今天...</span>
          </el-button>

          <p v-if="loading" class="loading-tip">
            <span class="loading-dot" />
            {{ loadingTips[tipIndex] }}
          </p>
        </div>

        <!-- 结果阶段 -->
        <div v-else key="result" class="result-section">
          <div class="result-header">
            <div class="result-badge" :class="`mode--${result.mode}`">
              <span class="badge-dot" />
              {{ modeLabel[result.mode] }}
            </div>
            <h1 class="result-title">
              今日方案<span class="gradient-text">已生成</span>
            </h1>
            <p class="result-date">{{ result.date }}</p>
          </div>

          <!-- 趋势分析 -->
          <div class="result-card card trend-card">
            <div class="card-tag">📈 TREND ANALYSIS</div>
            <p class="card-text">{{ result.trend_summary }}</p>
          </div>

          <!-- 训练 + 饮食 -->
          <div class="result-grid">
            <div class="result-card card">
              <div class="card-tag workout-tag">💪 WORKOUT PLAN</div>
              <pre class="card-text">{{ result.workout_plan }}</pre>
            </div>
            <div class="result-card card">
              <div class="card-tag diet-tag">🥗 NUTRITION</div>
              <pre class="card-text">{{ result.diet_plan }}</pre>
            </div>
          </div>

          <!-- 教练寄语 -->
          <div class="motivation-card card">
            <div class="motivation-bar" />
            <div class="motivation-inner">
              <span class="motivation-icon">✨</span>
              <div>
                <div class="motivation-from">FROM YOUR AI COACH</div>
                <p class="motivation-text">{{ result.motivation_message }}</p>
              </div>
            </div>
          </div>

          <!-- 重新生成 -->
          <div class="regenerate-section card">
            <div class="regenerate-inner">
              <div class="regenerate-left">
                <div class="regenerate-title">⚡ 对方案不满意？</div>
                <div class="regenerate-sub">
                  <span v-if="pendingDirectives">
                    🤖 小助手已记录你的调整意见，点击重新生成
                  </span>
                  <span v-else>
                    点击右下角 🤖 小助手告诉我你的想法，或直接重新生成
                  </span>
                </div>
              </div>
              <el-button
                class="regen-btn"
                type="primary"
                :loading="regenerating"
                @click="handleRegenerate"
              >
                <span v-if="!regenerating">{{ pendingDirectives ? '✨ 个性化重生' : '🔄 重新生成' }}</span>
                <span v-else>重写中...</span>
              </el-button>
            </div>
          </div>

          <!-- 操作按钮 -->
          <div class="result-actions">
            <el-button size="large" @click="router.push('/')">回到主页</el-button>
            <el-button size="large" @click="router.push('/history')">查看历史</el-button>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { dailyApi, profileApi } from '@/api'
import type { CheckinResponse } from '@/api/types'
import { useAssistantStore } from '@/stores/assistant'

const router = useRouter()
const assistantStore = useAssistantStore()
const loading = ref(false)
const regenerating = ref(false)
const result = ref<CheckinResponse | null>(null)
const tipIndex = ref(0)
const pendingDirectives = computed(() => assistantStore.pendingDirectives)
let tipTimer: ReturnType<typeof setInterval>

const today = computed(() => {
  const d = new Date()
  return `${d.getFullYear()}.${String(d.getMonth() + 1).padStart(2, '0')}.${String(d.getDate()).padStart(2, '0')}`
})

const loadingTips = [
  '正在分析你的体重趋势...',
  'Analyst Agent 检测是否停滞...',
  'Orchestrator 仲裁决策...',
  'Coach Agent 生成训练计划...',
  'Diet Agent 计算热量方案...',
  'Mental Agent 准备激励消息...',
]

const moods: { value: 'good' | 'neutral' | 'tired'; emoji: string; label: string }[] = [
  { value: 'good', emoji: '😄', label: '状态很好' },
  { value: 'neutral', emoji: '😐', label: '一般般' },
  { value: 'tired', emoji: '😴', label: '有点累' },
]

const modeLabel: Record<string, string> = {
  normal: 'NORMAL MODE',
  aggressive: 'AGGRESSIVE MODE',
  conservative: 'CONSERVATIVE MODE',
}

const form = reactive({
  weight_kg: 75.0,
  steps: 8000,
  calories_intake: 2000,
  workout_done: true,
  mood: 'neutral' as 'good' | 'neutral' | 'tired',
})

function adjust(field: 'weight_kg' | 'calories_intake', delta: number) {
  if (field === 'weight_kg') {
    form.weight_kg = Math.max(30, Math.round((form.weight_kg + delta) * 10) / 10)
  } else {
    form.calories_intake = Math.max(0, form.calories_intake + delta)
  }
}

async function handleCheckin() {
  loading.value = true
  tipIndex.value = 0
  tipTimer = setInterval(() => {
    tipIndex.value = (tipIndex.value + 1) % loadingTips.length
  }, 3000)

  try {
    const res = await dailyApi.checkin(form)
    result.value = res.data
    assistantStore.setCurrentPlan(res.data)
    localStorage.setItem('lastCheckin', JSON.stringify(res.data))
    ElMessage({ message: '方案生成成功 ⚡', type: 'success' })
  } catch (e: any) {
    ElMessage({ message: e.response?.data?.detail || '打卡失败，请重试', type: 'error' })
  } finally {
    loading.value = false
    clearInterval(tipTimer)
  }
}

async function handleRegenerate() {
  regenerating.value = true
  tipIndex.value = 0
  tipTimer = setInterval(() => {
    tipIndex.value = (tipIndex.value + 1) % loadingTips.length
  }, 3000)

  try {
    const directives = assistantStore.pendingDirectives ?? null
    const res = await dailyApi.regenerate({ ...form, directives })
    result.value = res.data
    assistantStore.setCurrentPlan(res.data)
    assistantStore.clearDirectives()
    localStorage.setItem('lastCheckin', JSON.stringify(res.data))
    ElMessage({ message: directives ? '个性化方案已重新生成 ✨' : '方案已重新生成 🔄', type: 'success' })
  } catch (e: any) {
    ElMessage({ message: e.response?.data?.detail || '重新生成失败，请重试', type: 'error' })
  } finally {
    regenerating.value = false
    clearInterval(tipTimer)
  }
}

onMounted(async () => {
  try {
    const res = await profileApi.get()
    form.weight_kg = res.data.weight_kg
  } catch {}
})

onUnmounted(() => clearInterval(tipTimer))
</script>

<style scoped>
.checkin-page {
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
  max-width: 800px;
  margin: 0 auto;
  padding: 48px 24px 96px;
  position: relative;
  z-index: 2;
}

.form-header { margin-bottom: 36px; animation: fade-up 0.6s var(--ease-out); }
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

/* 输入网格 */
.input-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-bottom: 16px;
}
.input-card {
  padding: 24px;
  animation: fade-up 0.6s var(--ease-out) both;
}
.input-card:nth-child(1) { animation-delay: 0.05s; }
.input-card:nth-child(2) { animation-delay: 0.1s; }
.input-card:nth-child(3) { animation-delay: 0.15s; }
.input-card:nth-child(4) { animation-delay: 0.2s; }
.input-label {
  display: flex;
  align-items: center;
  gap: 10px;
  font-family: var(--font-mono);
  font-size: 11px;
  font-weight: 600;
  color: var(--text-muted);
  letter-spacing: 0.16em;
  margin-bottom: 18px;
}
.input-icon { font-size: 18px; }

/* 体重调节 */
.weight-input {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}
.adj-btn {
  width: 38px; height: 38px;
  border-radius: 50%;
  border: 1.5px solid var(--border);
  background: rgba(255,255,255,0.04);
  font-size: 18px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--text-secondary);
  transition: var(--transition);
  flex-shrink: 0;
}
.adj-btn:hover {
  border-color: var(--green);
  color: var(--green-bright);
  background: rgba(16,219,161,0.1);
  box-shadow: 0 0 16px rgba(16,219,161,0.4);
  transform: scale(1.1);
}
.weight-display { text-align: center; }
.weight-num {
  font-family: var(--font-display);
  font-size: 38px;
  font-weight: 800;
  background-size: 200% auto;
  animation: gradient-shift 4s ease-in-out infinite;
}
.weight-unit {
  font-family: var(--font-mono);
  font-size: 13px;
  color: var(--text-muted);
  margin-left: 6px;
  font-weight: 600;
}

/* 步数滑块 */
.steps-display { width: 100%; }
.steps-num {
  font-family: var(--font-display);
  font-size: 26px;
  font-weight: 700;
  margin-top: 10px;
  background-size: 200% auto;
  animation: gradient-shift 4s ease-in-out infinite;
}
.unit-sm {
  font-family: var(--font-mono);
  font-size: 12px;
  color: var(--text-muted);
  margin-left: 4px;
  -webkit-text-fill-color: var(--text-muted);
}

/* 训练开关 */
.workout-toggle {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}
.toggle-option {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 14px;
  border-radius: var(--radius-sm);
  border: 1.5px solid var(--border);
  background: rgba(255,255,255,0.03);
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  color: var(--text-secondary);
  transition: var(--transition);
}
.toggle-option:hover { border-color: var(--border-bright); }
.toggle-option.active {
  border-color: transparent;
  background: rgba(16,219,161,0.12);
  color: var(--green-bright);
  box-shadow: 0 0 0 1.5px var(--green), 0 0 20px rgba(16,219,161,0.4);
}
.toggle-option--no.active {
  background: rgba(255,77,141,0.12);
  color: var(--pink-bright);
  box-shadow: 0 0 0 1.5px var(--pink), 0 0 20px rgba(255,77,141,0.4);
}

/* 情绪 */
.mood-section {
  padding: 28px;
  margin-bottom: 28px;
  animation: fade-up 0.6s var(--ease-out) 0.25s both;
}
.mood-options {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}
.mood-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  padding: 22px 12px;
  border-radius: var(--radius-sm);
  border: 1.5px solid var(--border);
  background: rgba(255,255,255,0.03);
  cursor: pointer;
  transition: var(--transition);
}
.mood-card:hover {
  border-color: var(--border-bright);
  transform: translateY(-3px);
  box-shadow: 0 8px 24px rgba(16,219,161,0.15);
}
.mood-card.selected {
  border-color: transparent;
  background: rgba(16,219,161,0.1);
  box-shadow: 0 0 0 1.5px var(--green), 0 0 24px rgba(16,219,161,0.4);
}
.mood-emoji {
  font-size: 34px;
  transition: transform 0.3s ease;
}
.mood-card.selected .mood-emoji { transform: scale(1.1); }
.mood-label {
  font-size: 13px;
  font-weight: 600;
  color: var(--text-secondary);
}
.mood-card.selected .mood-label { color: var(--green-bright); }

.submit-btn {
  width: 100%;
  height: 60px !important;
  font-size: 16px !important;
  font-weight: 700 !important;
  letter-spacing: 0.04em;
}
.loading-tip {
  text-align: center;
  margin-top: 18px;
  color: var(--text-muted);
  font-family: var(--font-mono);
  font-size: 12px;
  letter-spacing: 0.08em;
  animation: pulse-soft 2s ease-in-out infinite;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}
.loading-dot {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: var(--green-bright);
  box-shadow: 0 0 10px var(--green-bright);
  animation: pulse-soft 1s ease-in-out infinite;
}

/* 结果 */
.result-header {
  text-align: center;
  margin-bottom: 36px;
  animation: fade-up 0.6s var(--ease-out);
}
.result-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 6px 16px;
  border-radius: var(--radius-pill);
  font-family: var(--font-mono);
  font-size: 11px;
  font-weight: 700;
  margin-bottom: 16px;
  letter-spacing: 0.16em;
  border: 1px solid;
}
.badge-dot {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: currentColor;
  box-shadow: 0 0 8px currentColor;
  animation: pulse-soft 1.4s ease-in-out infinite;
}
.mode--normal { background: rgba(16,219,161,0.1); color: var(--green-bright); border-color: var(--border-bright); }
.mode--aggressive { background: rgba(255,77,141,0.1); color: var(--pink-bright); border-color: var(--border-pink); }
.mode--conservative { background: rgba(192,132,252,0.1); color: var(--neon-purple); border-color: rgba(192,132,252,0.4); }
.result-title {
  font-family: var(--font-display);
  font-size: 44px;
  font-weight: 800;
  color: var(--text-primary);
  margin-bottom: 8px;
  letter-spacing: -0.02em;
}
.result-date {
  color: var(--text-muted);
  font-family: var(--font-mono);
  font-size: 13px;
  letter-spacing: 0.08em;
}

.trend-card {
  padding: 28px;
  margin-bottom: 16px;
  animation: fade-up 0.6s var(--ease-out) 0.1s both;
}
.result-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-bottom: 16px;
  animation: fade-up 0.6s var(--ease-out) 0.2s both;
}
.result-card { padding: 28px; }
.card-tag {
  font-family: var(--font-mono);
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.18em;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 14px;
  animation: gradient-shift 4s ease-in-out infinite;
}
.workout-tag { animation-delay: 0.5s; }
.diet-tag { animation-delay: 1s; }
.card-text {
  font-size: 13px;
  color: var(--text-secondary);
  line-height: 1.85;
  white-space: pre-wrap;
  font-family: var(--font-body);
  max-height: 260px;
  overflow-y: auto;
}

.motivation-card {
  padding: 28px;
  background: linear-gradient(135deg, rgba(16,219,161,0.08), rgba(255,77,141,0.08)) !important;
  border-color: var(--border-bright) !important;
  margin-bottom: 16px;
  animation: fade-up 0.6s var(--ease-out) 0.3s both;
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
.motivation-inner {
  display: flex;
  align-items: flex-start;
  gap: 16px;
}
.motivation-icon {
  font-size: 26px;
  flex-shrink: 0;
  animation: pulse-soft 2s ease-in-out infinite;
}
.motivation-from {
  font-family: var(--font-mono);
  font-size: 11px;
  font-weight: 700;
  color: var(--green-bright);
  letter-spacing: 0.18em;
  margin-bottom: 8px;
}
.motivation-text {
  font-size: 15px;
  color: var(--text-secondary);
  line-height: 1.8;
  font-style: italic;
}

.regenerate-section {
  padding: 24px 28px;
  margin-bottom: 16px;
  animation: fade-up 0.6s var(--ease-out) 0.35s both;
  background: linear-gradient(135deg, rgba(192,132,252,0.06), rgba(255,77,141,0.06)) !important;
  border: 1px solid rgba(192,132,252,0.25) !important;
}
.regenerate-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}
.regenerate-title {
  font-family: var(--font-display);
  font-size: 16px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 6px;
}
.regenerate-sub {
  font-size: 12px;
  color: var(--text-muted);
  line-height: 1.5;
}
.regen-btn {
  flex-shrink: 0;
  height: 44px !important;
  padding: 0 22px !important;
  font-size: 13px !important;
  font-weight: 600 !important;
}

.result-actions {
  display: flex;
  gap: 12px;
  justify-content: center;
  animation: fade-up 0.6s var(--ease-out) 0.4s both;
}
.result-actions .el-button {
  height: 50px !important;
  padding: 0 32px !important;
  font-size: 14px !important;
}

@media (max-width: 768px) {
  .navbar { padding: 16px 20px; }
  .input-grid { grid-template-columns: 1fr; }
  .result-grid { grid-template-columns: 1fr; }
  .page-title { font-size: 36px; }
  .result-title { font-size: 32px; }
  .regenerate-inner { flex-direction: column; align-items: stretch; }
}
</style>
