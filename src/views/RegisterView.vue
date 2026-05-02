<template>
  <div class="auth-page">
    <!-- 极光层 -->
    <div class="aurora aurora--green" />
    <div class="aurora aurora--pink" />
    <div class="aurora aurora--purple" />
    <div class="grid-overlay" />
    <div class="scan-line" />

    <!-- 粒子层 -->
    <div class="particles">
      <span
        v-for="n in 16"
        :key="n"
        class="particle"
        :style="particleStyle(n)"
      />
    </div>

    <div class="register-wrap">
      <!-- Logo -->
      <div class="logo" @click="router.push('/login')">
        <div class="logo-orb">
          <span class="logo-flame">🔥</span>
          <span class="logo-ring" />
        </div>
        <span class="logo-name">FitGenie</span>
      </div>

      <div class="register-card glow-border">
        <div class="card-shine" />

        <!-- 步骤指示器 -->
        <div class="steps">
          <div
            v-for="(s, i) in stepLabels"
            :key="i"
            class="step"
            :class="{ active: step === i + 1, done: step > i + 1 }"
          >
            <div class="step-dot">
              <span v-if="step > i + 1">✓</span>
              <span v-else>{{ i + 1 }}</span>
            </div>
            <span class="step-label">{{ s }}</span>
          </div>
          <div class="step-line">
            <div class="step-line-fill" :style="{ width: step === 1 ? '0%' : '100%' }" />
          </div>
        </div>

        <!-- Step 1：账号信息 -->
        <transition name="step-slide" mode="out-in">
          <div v-if="step === 1" key="step1" class="step-content">
            <div class="step-header">
              <div class="step-tag">
                <span class="tag-dot" />
                STEP 01 / 02
              </div>
              <h2 class="step-title">创建账号</h2>
              <p class="step-sub">几个字段，开启你的减脂之旅</p>
            </div>

            <el-form
              ref="form1Ref"
              :model="form"
              :rules="rules1"
              label-position="top"
            >
              <el-form-item label="USERNAME" prop="username">
                <el-input
                  v-model="form.username"
                  placeholder="设置一个独特的用户名"
                  size="large"
                  prefix-icon="User"
                />
              </el-form-item>
              <el-form-item label="NICKNAME" prop="name">
                <el-input
                  v-model="form.name"
                  placeholder="想被怎么称呼？"
                  size="large"
                  prefix-icon="StarFilled"
                />
              </el-form-item>
              <el-form-item label="PASSWORD" prop="password">
                <el-input
                  v-model="form.password"
                  type="password"
                  placeholder="至少 6 位"
                  size="large"
                  prefix-icon="Lock"
                  show-password
                />
              </el-form-item>

              <el-button
                class="submit-btn"
                type="primary"
                size="large"
                @click="nextStep"
              >
                下一步 →
              </el-button>
            </el-form>
          </div>

          <!-- Step 2：身体数据 -->
          <div v-else key="step2" class="step-content">
            <div class="step-header">
              <div class="step-tag">
                <span class="tag-dot" />
                STEP 02 / 02
              </div>
              <h2 class="step-title">身体数据</h2>
              <p class="step-sub">AI 将根据这些数据为你量身定制</p>
            </div>

            <el-form
              ref="form2Ref"
              :model="form"
              :rules="rules2"
              label-position="top"
            >
              <div class="form-row">
                <el-form-item label="AGE" prop="age">
                  <el-input
                    v-model.number="form.age"
                    placeholder="岁"
                    size="large"
                    type="number"
                  />
                </el-form-item>
                <el-form-item label="HEIGHT (CM)" prop="height_cm">
                  <el-input
                    v-model.number="form.height_cm"
                    placeholder="cm"
                    size="large"
                    type="number"
                  />
                </el-form-item>
              </div>

              <el-form-item label="WEIGHT (KG)" prop="weight_kg">
                <el-input
                  v-model.number="form.weight_kg"
                  placeholder="kg"
                  size="large"
                  type="number"
                />
              </el-form-item>

              <el-form-item label="GOAL" prop="goal">
                <div class="goal-options">
                  <div
                    v-for="g in goalOptions"
                    :key="g.value"
                    class="goal-card"
                    :class="{ selected: form.goal === g.value }"
                    @click="form.goal = g.value"
                  >
                    <span class="goal-icon">{{ g.icon }}</span>
                    <span class="goal-label">{{ g.label }}</span>
                  </div>
                </div>
              </el-form-item>

              <el-form-item label="ACTIVITY LEVEL" prop="activity_level">
                <el-select
                  v-model="form.activity_level"
                  placeholder="选择你的日常活动水平"
                  size="large"
                  style="width: 100%"
                >
                  <el-option label="久坐（几乎不运动）" value="sedentary" />
                  <el-option label="轻度（每周1-3次运动）" value="light" />
                  <el-option label="中度（每周3-5次运动）" value="moderate" />
                  <el-option label="高强度（每天运动）" value="active" />
                </el-select>
              </el-form-item>

              <el-form-item label="DIETARY PREFERENCE" prop="dietary_pref">
                <el-select
                  v-model="form.dietary_pref"
                  placeholder="选择饮食偏好"
                  size="large"
                  style="width: 100%"
                >
                  <el-option label="无限制" value="no restriction" />
                  <el-option label="素食" value="vegetarian" />
                  <el-option label="纯素" value="vegan" />
                  <el-option label="低碳水" value="low_carb" />
                </el-select>
              </el-form-item>

              <div class="btn-row">
                <el-button size="large" @click="step = 1">← 上一步</el-button>
                <el-button
                  class="submit-btn-sm"
                  type="primary"
                  size="large"
                  :loading="loading"
                  @click="handleRegister"
                >
                  <span v-if="!loading">⚡ 开始燃烧</span>
                  <span v-else>创建中...</span>
                </el-button>
              </div>
            </el-form>
          </div>
        </transition>
      </div>

      <p class="footnote">
        已有账号？
        <router-link to="/login" class="link gradient-text">立即登录</router-link>
      </p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { useAuthStore } from '@/stores/auth'

const router = useRouter()
const authStore = useAuthStore()
const step = ref(1)
const loading = ref(false)
const form1Ref = ref()
const form2Ref = ref()

const stepLabels = ['账号信息', '身体数据']

const goalOptions = [
  { value: 'lose_fat', icon: '🔥', label: '减脂' },
  { value: 'build_muscle', icon: '💪', label: '增肌' },
  { value: 'maintain', icon: '⚖️', label: '维持' },
]

const form = reactive({
  username: '',
  password: '',
  name: '',
  age: null as number | null,
  weight_kg: null as number | null,
  height_cm: null as number | null,
  goal: 'lose_fat',
  activity_level: '',
  dietary_pref: '',
})

const rules1 = {
  username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
  name: [{ required: true, message: '请输入昵称', trigger: 'blur' }],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, message: '至少6位', trigger: 'blur' },
  ],
}

const rules2 = {
  age: [{ required: true, message: '请输入年龄', trigger: 'blur' }],
  weight_kg: [{ required: true, message: '请输入体重', trigger: 'blur' }],
  height_cm: [{ required: true, message: '请输入身高', trigger: 'blur' }],
  activity_level: [{ required: true, message: '请选择活动水平', trigger: 'change' }],
  dietary_pref: [{ required: true, message: '请选择饮食偏好', trigger: 'change' }],
}

function particleStyle(n: number) {
  const left = (n * 67) % 100
  const delay = (n * 0.7) % 8
  const duration = 8 + (n * 1.1) % 10
  const size = 2 + (n % 4)
  const hue = n % 2 === 0 ? '#10dba1' : '#ff4d8d'
  return {
    left: `${left}%`,
    animationDelay: `${delay}s`,
    animationDuration: `${duration}s`,
    width: `${size}px`,
    height: `${size}px`,
    background: hue,
    boxShadow: `0 0 ${size * 4}px ${hue}`,
  }
}

async function nextStep() {
  await form1Ref.value?.validate()
  step.value = 2
}

async function handleRegister() {
  await form2Ref.value?.validate()
  loading.value = true
  try {
    await authStore.register(form as any)
    ElMessage({ message: '注册成功，欢迎加入 FitGenie 🎉', type: 'success' })
    router.push('/')
  } catch (e: any) {
    ElMessage({ message: e.response?.data?.detail || '注册失败', type: 'error' })
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.auth-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px 24px;
  position: relative;
  overflow: hidden;
  z-index: 0;
}

/* 极光 */
.aurora {
  position: fixed;
  border-radius: 50%;
  filter: blur(90px);
  pointer-events: none;
  opacity: 0.5;
  mix-blend-mode: screen;
}
.aurora--green {
  width: 700px; height: 700px;
  background: radial-gradient(circle, #10dba1, transparent 60%);
  top: -200px; right: -200px;
  animation: aurora 20s linear infinite;
}
.aurora--pink {
  width: 600px; height: 600px;
  background: radial-gradient(circle, #ff4d8d, transparent 60%);
  bottom: -180px; left: -180px;
  animation: aurora 16s linear infinite reverse;
}
.aurora--purple {
  width: 480px; height: 480px;
  background: radial-gradient(circle, #c084fc, transparent 60%);
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  animation: float-slow 14s ease-in-out infinite;
  opacity: 0.3;
}

.grid-overlay {
  position: fixed; inset: 0;
  background-image:
    linear-gradient(rgba(16,219,161,0.05) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,77,141,0.05) 1px, transparent 1px);
  background-size: 40px 40px;
  pointer-events: none;
  mask-image: radial-gradient(circle at center, black 30%, transparent 80%);
}

.scan-line {
  position: fixed;
  left: 0; right: 0;
  height: 200px;
  background: linear-gradient(to bottom, transparent, rgba(255,77,141,0.08), transparent);
  pointer-events: none;
  animation: scan-line 9s linear infinite;
}

.particles {
  position: fixed; inset: 0;
  pointer-events: none;
  overflow: hidden;
}
.particle {
  position: absolute;
  bottom: -10px;
  border-radius: 50%;
  opacity: 0;
  animation: particle-rise linear infinite;
}

.register-wrap {
  width: 100%;
  max-width: 540px;
  position: relative;
  z-index: 10;
  animation: scale-in 0.7s var(--ease-out);
}

.logo {
  display: flex;
  align-items: center;
  gap: 14px;
  margin-bottom: 28px;
  cursor: pointer;
  user-select: none;
}
.logo-orb {
  width: 40px; height: 40px;
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
  font-size: 18px;
  animation: flame-flicker 2s ease-in-out infinite;
}
.logo-ring {
  position: absolute;
  inset: -4px;
  border-radius: 50%;
  border: 2px solid var(--green-bright);
  opacity: 0;
  animation: ring-pulse 2.5s ease-out infinite;
}
.logo-name {
  font-family: var(--font-display);
  font-size: 22px;
  font-weight: 700;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradient-shift 5s ease-in-out infinite;
}

.register-card {
  padding: 44px;
  background: var(--bg-glass);
  backdrop-filter: blur(24px) saturate(180%);
  -webkit-backdrop-filter: blur(24px) saturate(180%);
  position: relative;
  overflow: hidden;
}
.card-shine {
  position: absolute;
  top: 0; left: -100%;
  width: 60%; height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,77,141,0.06), transparent);
  animation: shimmer 4s ease-in-out infinite;
  pointer-events: none;
}

/* 步骤条 */
.steps {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 36px;
  position: relative;
}
.step {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 1;
  position: relative;
  z-index: 1;
}
.step-dot {
  width: 32px; height: 32px;
  border-radius: 50%;
  border: 1.5px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
  font-weight: 700;
  font-family: var(--font-mono);
  color: var(--text-muted);
  background: var(--bg-elevated);
  transition: var(--transition);
  flex-shrink: 0;
}
.step.active .step-dot {
  border-color: var(--green);
  color: var(--green-bright);
  background: rgba(16,219,161,0.1);
  box-shadow: 0 0 0 4px rgba(16,219,161,0.15), 0 0 20px rgba(16,219,161,0.5);
}
.step.done .step-dot {
  background: var(--gradient-primary);
  border-color: transparent;
  color: white;
  box-shadow: var(--glow-mix);
}
.step-label {
  font-family: var(--font-mono);
  font-size: 11px;
  letter-spacing: 0.12em;
  color: var(--text-muted);
  font-weight: 600;
  transition: var(--transition);
}
.step.active .step-label { color: var(--green-bright); }
.step.done .step-label { color: var(--text-secondary); }
.step-line {
  position: absolute;
  top: 16px; left: 40px;
  right: 40px;
  height: 2px;
  background: var(--border);
  z-index: 0;
  border-radius: 2px;
  overflow: hidden;
}
.step-line-fill {
  height: 100%;
  background: var(--gradient-primary);
  transition: width 0.6s var(--ease-out);
  box-shadow: var(--glow-mix);
}

.step-header { margin-bottom: 28px; }
.step-tag {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-family: var(--font-mono);
  font-size: 11px;
  letter-spacing: 0.2em;
  color: var(--green);
  margin-bottom: 14px;
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
.step-title {
  font-family: var(--font-display);
  font-size: 32px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 8px;
  letter-spacing: -0.02em;
}
.step-sub {
  color: var(--text-secondary);
  font-size: 14px;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

/* 目标选择 */
.goal-options {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  width: 100%;
}
.goal-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 18px 8px;
  border-radius: var(--radius-sm);
  border: 1.5px solid var(--border);
  background: rgba(255,255,255,0.03);
  cursor: pointer;
  transition: var(--transition);
  position: relative;
  overflow: hidden;
}
.goal-card::before {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--gradient-primary);
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: 0;
}
.goal-card:hover {
  border-color: var(--border-bright);
  transform: translateY(-3px);
  box-shadow: 0 8px 24px rgba(16,219,161,0.2);
}
.goal-card.selected {
  border-color: transparent;
  background: rgba(16,219,161,0.1);
  box-shadow: 0 0 0 1.5px var(--green), 0 0 24px rgba(16,219,161,0.4);
}
.goal-card.selected::before { opacity: 0.08; }
.goal-icon {
  font-size: 26px;
  position: relative;
  z-index: 1;
  transition: transform 0.3s ease;
}
.goal-card.selected .goal-icon {
  animation: flame-flicker 2s ease-in-out infinite;
}
.goal-label {
  font-size: 13px;
  font-weight: 600;
  color: var(--text-secondary);
  position: relative;
  z-index: 1;
}
.goal-card.selected .goal-label { color: var(--green-bright); }

.submit-btn {
  width: 100%;
  height: 56px !important;
  font-size: 15px !important;
  margin-top: 8px;
}
.btn-row {
  display: flex;
  gap: 12px;
  margin-top: 8px;
}
.btn-row .el-button { height: 52px !important; }
.submit-btn-sm {
  flex: 1;
  font-size: 15px !important;
}

.footnote {
  text-align: center;
  margin-top: 24px;
  color: var(--text-secondary);
  font-size: 14px;
}
.link {
  text-decoration: none;
  font-weight: 600;
}

/* 步骤切换动画 */
.step-slide-enter-active, .step-slide-leave-active {
  transition: all 0.4s var(--ease-out);
}
.step-slide-enter-from {
  opacity: 0; transform: translateX(40px);
}
.step-slide-leave-to {
  opacity: 0; transform: translateX(-40px);
}

@media (max-width: 480px) {
  .register-card { padding: 32px 24px; }
  .step-title { font-size: 26px; }
  .form-row { grid-template-columns: 1fr; }
}
</style>
