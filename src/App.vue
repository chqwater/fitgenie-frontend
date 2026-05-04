<template>
  <router-view v-slot="{ Component }">
    <transition name="fade" mode="out-in">
      <component :is="Component" />
    </transition>
  </router-view>

  <!-- 全局小助手面板（只在登录状态下显示） -->
  <AssistantPanel
    v-if="isLoggedIn"
    :current-plan="currentPlan ? (currentPlan as unknown as Record<string, unknown>) : null"
    @directives-updated="onDirectivesUpdated"
  />

  <!-- 全局页脚 -->
  <footer class="app-footer">
    <span class="footer-line" />
    <span class="footer-text">
      © {{ year }} <span class="footer-brand">FitGenie</span> · 陈浩泉 Haoquan Chen · MSc, Imperial College London
    </span>
    <span class="footer-line" />
  </footer>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import AssistantPanel from '@/components/AssistantPanel.vue'
import { useAssistantStore } from '@/stores/assistant'
import type { AssistantDirectives } from '@/api/types'

const assistantStore = useAssistantStore()
const currentPlan = computed(() => assistantStore.currentPlan)
const isLoggedIn = computed(() => !!localStorage.getItem('token'))
const year = new Date().getFullYear()

function onDirectivesUpdated(d: AssistantDirectives) {
  assistantStore.setPendingDirectives(d)
}
</script>

<style scoped>
.app-footer {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  padding: 28px 24px 32px;
  font-family: var(--font-mono, monospace);
  font-size: 11px;
  color: var(--text-muted, #6b7691);
  letter-spacing: 0.12em;
  background: transparent;
  position: relative;
  z-index: 10;
}
.footer-line {
  flex: 0 1 80px;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--border, rgba(255,255,255,0.1)), transparent);
}
.footer-text {
  white-space: nowrap;
}
.footer-brand {
  background: var(--gradient-text, linear-gradient(135deg, #00ffaa, #ff5ea2));
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 700;
  animation: gradient-shift 5s ease-in-out infinite;
}

@media (max-width: 640px) {
  .app-footer {
    font-size: 10px;
    padding: 20px 12px 24px;
    gap: 8px;
  }
  .footer-line { flex: 0 1 30px; }
  .footer-text { white-space: normal; text-align: center; line-height: 1.6; }
}
</style>
