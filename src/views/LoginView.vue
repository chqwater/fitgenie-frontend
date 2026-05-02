<template>
  <div class="auth-page">
    <!-- 极光层 -->
    <div class="aurora aurora--green" />
    <div class="aurora aurora--pink" />
    <div class="aurora aurora--purple" />

    <!-- 网格层 -->
    <div class="grid-overlay" />
    <div class="scan-line" />

    <!-- 粒子层 -->
    <div class="particles">
      <span
        v-for="n in 18"
        :key="n"
        class="particle"
        :style="particleStyle(n)"
      />
    </div>

    <div class="auth-wrap">
      <!-- 左侧品牌区 -->
      <div class="brand-side">
        <div class="brand-inner">
          <div class="logo">
            <div class="logo-orb">
              <span class="logo-flame">🔥</span>
              <span class="logo-ring" />
            </div>
            <span class="logo-name">FitGenie</span>
          </div>

          <h1 class="brand-title">
            <span class="line">Burn fat.</span>
            <span class="line gradient-text">Forge limits.</span>
            <span class="line">Become legend.</span>
          </h1>

          <p class="brand-desc">
            <span class="cursor">▍</span>
            多智能体驱动的减脂引擎 · 每一天，AI 为你重写训练与饮食的最优解。
          </p>

          <div class="brand-stats">
            <div class="stat" v-for="(s, i) in stats" :key="s.label" :style="{ animationDelay: `${0.6 + i * 0.15}s` }">
              <span class="stat-num gradient-text">{{ s.num }}</span>
              <span class="stat-label">{{ s.label }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 右侧表单区 -->
      <div class="form-side">
        <div class="form-card glow-border">
          <div class="card-shine" />

          <div class="form-header">
            <div class="form-tag">
              <span class="tag-dot" />
              SECURE LOGIN
            </div>
            <h2 class="form-title">Welcome Back</h2>
            <p class="form-sub">登录以解锁今日 AI 训练方案</p>
          </div>

          <el-form
            ref="formRef"
            :model="form"
            :rules="rules"
            label-position="top"
          >
            <el-form-item label="USERNAME" prop="username">
              <el-input
                v-model="form.username"
                placeholder="输入用户名"
                size="large"
                prefix-icon="User"
              />
            </el-form-item>

            <el-form-item label="PASSWORD" prop="password">
              <el-input
                v-model="form.password"
                type="password"
                placeholder="输入密码"
                size="large"
                prefix-icon="Lock"
                show-password
                @keyup.enter="handleLogin"
              />
            </el-form-item>

            <el-button
              class="submit-btn"
              type="primary"
              size="large"
              :loading="loading"
              @click="handleLogin"
            >
              <span v-if="!loading">⚡ 进入今日训练</span>
              <span v-else>验证身份中...</span>
            </el-button>
          </el-form>

          <div class="divider">
            <span>NEW HERE?</span>
          </div>

          <router-link to="/register" class="register-link">
            <el-button class="register-btn" size="large">
              创建账号 →
            </el-button>
          </router-link>
        </div>

        <p class="footnote">
          <span class="dot-pulse" /> 科学训练 · 智能饮食 · 持续燃烧
        </p>
      </div>
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
const formRef = ref()
const loading = ref(false)

const stats = [
  { num: '5+', label: 'AI AGENTS' },
  { num: '∞', label: 'WORKOUTS' },
  { num: '24/7', label: 'TRACKING' },
]

const form = reactive({ username: '', password: '' })
const rules = {
  username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
  password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
}

function particleStyle(n: number) {
  const left = (n * 53) % 100
  const delay = (n * 0.6) % 8
  const duration = 8 + (n * 1.3) % 10
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

async function handleLogin() {
  await formRef.value?.validate()
  loading.value = true
  try {
    await authStore.login(form)
    ElMessage({ message: '登录成功 ⚡', type: 'success' })
    router.push('/')
  } catch (e: any) {
    ElMessage({ message: e.response?.data?.detail || '用户名或密码错误', type: 'error' })
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
  position: relative;
  overflow: hidden;
  padding: 24px;
  z-index: 0;
}

/* ── 极光 ── */
.aurora {
  position: fixed;
  border-radius: 50%;
  filter: blur(90px);
  pointer-events: none;
  opacity: 0.55;
  mix-blend-mode: screen;
}
.aurora--green {
  width: 720px; height: 720px;
  background: radial-gradient(circle, #10dba1, transparent 60%);
  top: -240px; left: -180px;
  animation: aurora 22s linear infinite;
}
.aurora--pink {
  width: 640px; height: 640px;
  background: radial-gradient(circle, #ff4d8d, transparent 60%);
  bottom: -200px; right: -160px;
  animation: aurora 18s linear infinite reverse;
}
.aurora--purple {
  width: 520px; height: 520px;
  background: radial-gradient(circle, #c084fc, transparent 60%);
  top: 40%; left: 50%;
  transform: translate(-50%, -50%);
  animation: float-slow 16s ease-in-out infinite;
  opacity: 0.35;
}

/* 网格 */
.grid-overlay {
  position: fixed; inset: 0;
  background-image:
    linear-gradient(rgba(16, 219, 161, 0.05) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 77, 141, 0.05) 1px, transparent 1px);
  background-size: 40px 40px;
  pointer-events: none;
  mask-image: radial-gradient(circle at center, black 30%, transparent 80%);
}

/* 扫描线 */
.scan-line {
  position: fixed;
  left: 0; right: 0;
  height: 200px;
  background: linear-gradient(to bottom, transparent, rgba(16, 219, 161, 0.08), transparent);
  pointer-events: none;
  animation: scan-line 8s linear infinite;
}

/* 粒子 */
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

.auth-wrap {
  display: flex;
  gap: 80px;
  align-items: center;
  max-width: 1080px;
  width: 100%;
  position: relative;
  z-index: 10;
}

/* ── 左侧品牌 ── */
.brand-side {
  flex: 1;
  animation: fade-up 0.9s var(--ease-out) both;
}
.logo {
  display: flex;
  align-items: center;
  gap: 14px;
  margin-bottom: 48px;
}
.logo-orb {
  width: 44px; height: 44px;
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
  font-size: 20px;
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
  font-size: 24px;
  font-weight: 700;
  letter-spacing: 0.02em;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradient-shift 5s ease-in-out infinite;
}

.brand-title {
  font-family: var(--font-display);
  font-size: 64px;
  line-height: 1.05;
  font-weight: 700;
  margin-bottom: 28px;
  letter-spacing: -0.02em;
}
.brand-title .line {
  display: block;
  color: var(--text-primary);
  opacity: 0;
  transform: translateY(20px);
  animation: fade-up 0.8s var(--ease-out) forwards;
}
.brand-title .line:nth-child(1) { animation-delay: 0.1s; }
.brand-title .line:nth-child(2) { animation-delay: 0.3s; background-size: 200% auto; animation: fade-up 0.8s var(--ease-out) 0.3s forwards, gradient-shift 5s ease-in-out 0.3s infinite; }
.brand-title .line:nth-child(3) { animation-delay: 0.5s; }

.brand-desc {
  color: var(--text-secondary);
  font-size: 16px;
  line-height: 1.75;
  max-width: 380px;
  margin-bottom: 56px;
  font-family: var(--font-mono);
  opacity: 0;
  animation: fade-up 0.7s var(--ease-out) 0.7s forwards;
}
.cursor {
  color: var(--green-bright);
  animation: pulse-soft 1s ease-in-out infinite;
  margin-right: 4px;
}

.brand-stats {
  display: flex;
  gap: 48px;
}
.stat {
  display: flex;
  flex-direction: column;
  gap: 6px;
  opacity: 0;
  animation: fade-up 0.6s var(--ease-out) forwards;
}
.stat-num {
  font-family: var(--font-display);
  font-size: 36px;
  font-weight: 800;
  line-height: 1;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradient-shift 5s ease-in-out infinite;
}
.stat-label {
  font-family: var(--font-mono);
  font-size: 11px;
  color: var(--text-muted);
  letter-spacing: 0.18em;
  font-weight: 600;
}

/* ── 右侧表单 ── */
.form-side {
  width: 420px;
  flex-shrink: 0;
  animation: scale-in 0.7s var(--ease-out);
}
.form-card {
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
  background: linear-gradient(90deg, transparent, rgba(0, 255, 170, 0.08), transparent);
  animation: shimmer 4s ease-in-out infinite;
  pointer-events: none;
}

.form-header {
  margin-bottom: 32px;
  position: relative;
}
.form-tag {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-family: var(--font-mono);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.2em;
  color: var(--green);
  margin-bottom: 14px;
  padding: 4px 12px;
  border: 1px solid var(--border-bright);
  border-radius: var(--radius-pill);
  background: rgba(16, 219, 161, 0.08);
}
.tag-dot {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: var(--green-bright);
  box-shadow: 0 0 8px var(--green-bright);
  animation: pulse-soft 1.4s ease-in-out infinite;
}
.form-title {
  font-family: var(--font-display);
  font-size: 36px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 8px;
  letter-spacing: -0.02em;
}
.form-sub {
  color: var(--text-secondary);
  font-size: 14px;
}

.submit-btn {
  width: 100%;
  height: 56px !important;
  font-size: 15px !important;
  font-weight: 600 !important;
  margin-top: 8px;
  letter-spacing: 0.04em;
}

.divider {
  text-align: center;
  margin: 24px 0;
  color: var(--text-muted);
  font-family: var(--font-mono);
  font-size: 11px;
  letter-spacing: 0.2em;
  position: relative;
}
.divider::before, .divider::after {
  content: '';
  position: absolute;
  top: 50%;
  width: 35%;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--border));
}
.divider::before { left: 0; background: linear-gradient(90deg, transparent, var(--border)); }
.divider::after  { right: 0; background: linear-gradient(90deg, var(--border), transparent); }

.register-link { display: block; }
.register-btn {
  width: 100%;
  height: 50px !important;
  font-size: 14px !important;
  font-weight: 500 !important;
}

.footnote {
  text-align: center;
  margin-top: 24px;
  color: var(--text-muted);
  font-family: var(--font-mono);
  font-size: 11px;
  letter-spacing: 0.16em;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}
.dot-pulse {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: var(--pink-bright);
  box-shadow: 0 0 10px var(--pink-bright);
  animation: pulse-soft 1.6s ease-in-out infinite;
}

@media (max-width: 920px) {
  .brand-side { display: none; }
  .form-side { width: 100%; max-width: 440px; }
}
@media (max-width: 480px) {
  .form-card { padding: 32px 24px; }
  .form-title { font-size: 28px; }
}
</style>
