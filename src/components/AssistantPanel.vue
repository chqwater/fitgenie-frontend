<template>
  <!-- 浮动按钮 -->
  <div class="assistant-fab" @click="toggle" :class="{ open: isOpen }">
    <span class="fab-icon">{{ isOpen ? '✕' : '🤖' }}</span>
    <span v-if="!isOpen && unreadBadge" class="fab-badge">{{ unreadBadge }}</span>
  </div>

  <!-- 面板 -->
  <transition name="panel-slide">
    <div v-if="isOpen" class="assistant-panel">
      <div class="panel-header">
        <div class="panel-title">
          <span class="panel-icon">🤖</span>
          <div>
            <div class="panel-name">FitGenie 小助手</div>
            <div class="panel-sub">随时告诉我你的想法</div>
          </div>
        </div>
        <div class="panel-actions">
          <span
            v-if="hasPendingDirectives"
            class="directive-badge"
            title="已有待应用的调整指令"
          >有调整意见</span>
          <button class="panel-close" @click="toggle">✕</button>
        </div>
      </div>

      <!-- 偏好标签 -->
      <div v-if="preferenceTags.length" class="pref-tags">
        <span class="pref-label">已记录偏好：</span>
        <span v-for="tag in preferenceTags" :key="tag" class="pref-tag">{{ tag }}</span>
      </div>

      <!-- 对话记录 -->
      <div class="chat-messages" ref="messagesList">
        <div v-if="messages.length === 0" class="chat-empty">
          <p>你好！我是你的 FitGenie 小助手 👋</p>
          <p>你可以告诉我你的喜好、身体状况，或者对今日计划的想法，我会帮你定制更适合你的方案。</p>
        </div>

        <div
          v-for="(msg, i) in messages"
          :key="i"
          class="message"
          :class="msg.role"
        >
          <div class="message-bubble">{{ msg.content }}</div>
          <div v-if="msg.directives" class="directive-hint">
            <span class="directive-icon">⚡</span>
            已记录调整指令，点击"重新生成计划"应用
          </div>
        </div>

        <div v-if="thinking" class="message assistant">
          <div class="message-bubble thinking">
            <span class="dot" /><span class="dot" /><span class="dot" />
          </div>
        </div>
      </div>

      <!-- 输入区 -->
      <div class="chat-input-area">
        <textarea
          v-model="inputText"
          class="chat-input"
          placeholder="说说你的想法..."
          rows="2"
          @keydown.enter.exact.prevent="send"
        />
        <button class="send-btn" :disabled="!inputText.trim() || thinking" @click="send">
          发送
        </button>
      </div>
    </div>
  </transition>
</template>

<script setup lang="ts">
import { ref, computed, watch, nextTick, onMounted } from 'vue'
import { assistantApi } from '@/api'
import type { AssistantDirectives, AssistantHistoryItem } from '@/api/types'

interface Message {
  role: 'user' | 'assistant'
  content: string
  directives?: AssistantDirectives | null
}

const props = defineProps<{
  currentPlan?: Record<string, unknown> | null
}>()

const emit = defineEmits<{
  (e: 'directives-updated', d: AssistantDirectives): void
}>()

const isOpen = ref(false)
const inputText = ref('')
const thinking = ref(false)
const messages = ref<Message[]>([])
const messagesList = ref<HTMLElement | null>(null)
const unreadBadge = ref(0)
const pendingDirectives = ref<AssistantDirectives | null>(null)
const preferences = ref<Record<string, string>>({})

const hasPendingDirectives = computed(() => !!pendingDirectives.value)

const preferenceTags = computed(() => {
  const prefs = preferences.value
  const tags: string[] = []
  if (prefs.workout_likes) tags.push(`💪 ${prefs.workout_likes}`)
  if (prefs.workout_dislikes) tags.push(`🚫 避免：${prefs.workout_dislikes}`)
  if (prefs.diet_restrictions) tags.push(`🥗 ${prefs.diet_restrictions}`)
  if (prefs.injury_notes) tags.push(`⚠️ ${prefs.injury_notes}`)
  return tags.slice(0, 3)
})

function toggle() {
  isOpen.value = !isOpen.value
  if (isOpen.value) {
    unreadBadge.value = 0
    nextTick(() => scrollToBottom())
  }
}

async function send() {
  const text = inputText.value.trim()
  if (!text || thinking.value) return

  messages.value.push({ role: 'user', content: text })
  inputText.value = ''
  thinking.value = true
  await nextTick()
  scrollToBottom()

  try {
    const res = await assistantApi.chat({
      message: text,
      current_plan: props.currentPlan ?? null,
    })
    const { reply, directives, preferences_updated } = res.data

    messages.value.push({
      role: 'assistant',
      content: reply,
      directives: directives ?? null,
    })

    if (directives && Object.keys(directives).length > 0) {
      pendingDirectives.value = directives
      emit('directives-updated', directives)
    }

    if (preferences_updated) {
      await loadPreferences()
    }
  } catch {
    messages.value.push({
      role: 'assistant',
      content: '抱歉，我暂时无法回应，请稍后再试。',
    })
  } finally {
    thinking.value = false
    if (!isOpen.value) unreadBadge.value++
    await nextTick()
    scrollToBottom()
  }
}

function scrollToBottom() {
  if (messagesList.value) {
    messagesList.value.scrollTop = messagesList.value.scrollHeight
  }
}

async function loadHistory() {
  try {
    const res = await assistantApi.history(20)
    messages.value = res.data.map((item: AssistantHistoryItem) => ({
      role: item.role as 'user' | 'assistant',
      content: item.content,
    }))
    await nextTick()
    scrollToBottom()
  } catch {
    // 静默失败
  }
}

async function loadPreferences() {
  try {
    const res = await assistantApi.preferences()
    preferences.value = res.data ?? {}
  } catch {
    // 静默失败
  }
}

// 向外暴露待应用指令（供 CheckinView 使用）
function getDirectives() {
  return pendingDirectives.value
}

function clearDirectives() {
  pendingDirectives.value = null
}

defineExpose({ getDirectives, clearDirectives, toggle })

onMounted(async () => {
  await Promise.all([loadHistory(), loadPreferences()])
})

// 计划变化时让助手知晓
watch(() => props.currentPlan, (plan) => {
  if (plan && isOpen.value) {
    // 不自动发消息，只是让下次对话带上 context
  }
})
</script>

<style scoped>
/* ── 浮动按钮 ── */
.assistant-fab {
  position: fixed;
  bottom: 32px;
  right: 32px;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: var(--gradient-primary);
  background-size: 200% auto;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 1000;
  box-shadow: var(--glow-mix);
  transition: transform 0.3s var(--ease-elastic), box-shadow 0.3s ease;
  user-select: none;
  animation: gradient-shift 5s ease-in-out infinite, pulse-glow 3s ease-in-out infinite;
}
.assistant-fab::before {
  content: '';
  position: absolute;
  inset: -4px;
  border-radius: 50%;
  border: 2px solid var(--green-bright);
  opacity: 0;
  animation: ring-pulse 2.5s ease-out infinite;
  pointer-events: none;
}
.assistant-fab:hover {
  transform: scale(1.12) rotate(8deg);
  box-shadow: 0 0 50px rgba(16,219,161,0.7), 0 0 100px rgba(255,77,141,0.4);
}
.assistant-fab.open {
  transform: scale(0.92) rotate(180deg);
}
.fab-icon {
  font-size: 24px;
  filter: drop-shadow(0 0 6px rgba(255,255,255,0.6));
}
.fab-badge {
  position: absolute;
  top: -2px;
  right: -2px;
  min-width: 20px;
  height: 20px;
  padding: 0 5px;
  border-radius: var(--radius-pill);
  background: var(--pink);
  color: white;
  font-size: 11px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid var(--bg-base);
  box-shadow: 0 0 12px rgba(255,77,141,0.6);
  animation: pulse-soft 1.4s ease-in-out infinite;
}

/* ── 面板 ── */
.assistant-panel {
  position: fixed;
  bottom: 110px;
  right: 32px;
  width: 380px;
  max-height: 560px;
  background: var(--bg-glass);
  backdrop-filter: blur(24px) saturate(180%);
  -webkit-backdrop-filter: blur(24px) saturate(180%);
  border-radius: var(--radius);
  box-shadow: var(--shadow-lg), 0 0 0 1px var(--border-bright);
  display: flex;
  flex-direction: column;
  z-index: 999;
  overflow: hidden;
  border: 1px solid var(--border);
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 18px 20px;
  background: linear-gradient(135deg, rgba(16,219,161,0.12), rgba(255,77,141,0.08));
  border-bottom: 1px solid var(--border);
  position: relative;
}
.panel-header::after {
  content: '';
  position: absolute;
  bottom: 0; left: 0; right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--green-bright), var(--pink-bright), transparent);
  opacity: 0.4;
}
.panel-title {
  display: flex;
  align-items: center;
  gap: 12px;
}
.panel-icon {
  font-size: 26px;
  filter: drop-shadow(0 0 8px rgba(0,255,170,0.5));
  animation: pulse-soft 2s ease-in-out infinite;
}
.panel-name {
  font-family: var(--font-display);
  font-size: 15px;
  font-weight: 700;
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradient-shift 4s ease-in-out infinite;
}
.panel-sub {
  font-family: var(--font-mono);
  font-size: 10px;
  color: var(--text-muted);
  letter-spacing: 0.1em;
  margin-top: 2px;
}
.panel-actions {
  display: flex;
  align-items: center;
  gap: 10px;
}
.directive-badge {
  font-family: var(--font-mono);
  font-size: 10px;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: var(--radius-pill);
  background: rgba(255,77,141,0.15);
  color: var(--pink-bright);
  border: 1px solid var(--border-pink);
  letter-spacing: 0.04em;
  animation: pulse-soft 1.6s ease-in-out infinite;
}
.panel-close {
  background: rgba(255,255,255,0.05);
  border: 1px solid var(--border);
  border-radius: 50%;
  cursor: pointer;
  color: var(--text-secondary);
  font-size: 13px;
  width: 26px;
  height: 26px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: var(--transition);
}
.panel-close:hover {
  color: var(--pink-bright);
  border-color: var(--border-pink);
  background: rgba(255,77,141,0.1);
  transform: rotate(90deg);
}

/* 偏好标签 */
.pref-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  padding: 10px 16px;
  background: rgba(16,219,161,0.04);
  border-bottom: 1px solid var(--border);
  align-items: center;
}
.pref-label {
  font-family: var(--font-mono);
  font-size: 10px;
  color: var(--text-muted);
  flex-shrink: 0;
  letter-spacing: 0.08em;
}
.pref-tag {
  font-size: 10px;
  padding: 3px 10px;
  border-radius: var(--radius-pill);
  background: rgba(16,219,161,0.12);
  color: var(--green-bright);
  font-weight: 600;
  border: 1px solid var(--border-bright);
}

/* 消息区 */
.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 18px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.chat-empty {
  color: var(--text-muted);
  font-size: 13px;
  line-height: 1.8;
  text-align: center;
  padding: 24px 8px;
}
.chat-empty p { margin-bottom: 10px; }
.chat-empty p:first-child {
  background: var(--gradient-text);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 700;
  font-size: 15px;
  animation: gradient-shift 4s ease-in-out infinite;
}

.message { display: flex; flex-direction: column; animation: fade-up 0.3s var(--ease-out); }
.message.user { align-items: flex-end; }
.message.assistant { align-items: flex-start; }

.message-bubble {
  max-width: 85%;
  padding: 12px 16px;
  border-radius: 16px;
  font-size: 13px;
  line-height: 1.65;
  white-space: pre-wrap;
}
.message.user .message-bubble {
  background: var(--gradient-primary);
  background-size: 200% auto;
  color: white;
  border-bottom-right-radius: 4px;
  box-shadow: 0 4px 16px rgba(255,77,141,0.3);
  animation: gradient-shift 5s ease-in-out infinite;
}
.message.assistant .message-bubble {
  background: rgba(255,255,255,0.05);
  color: var(--text-primary);
  border-bottom-left-radius: 4px;
  border: 1px solid var(--border);
  backdrop-filter: blur(10px);
}

/* 思考动画 */
.thinking {
  display: flex;
  gap: 5px;
  align-items: center;
  padding: 14px 18px;
}
.dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--green-bright);
  box-shadow: 0 0 6px var(--green-bright);
  animation: bounce 1.2s ease-in-out infinite;
}
.dot:nth-child(2) {
  animation-delay: 0.15s;
  background: var(--neon-purple);
  box-shadow: 0 0 6px var(--neon-purple);
}
.dot:nth-child(3) {
  animation-delay: 0.3s;
  background: var(--pink-bright);
  box-shadow: 0 0 6px var(--pink-bright);
}
@keyframes bounce {
  0%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-8px); }
}

.directive-hint {
  margin-top: 6px;
  font-family: var(--font-mono);
  font-size: 10px;
  color: var(--pink-bright);
  display: flex;
  align-items: center;
  gap: 6px;
  padding-left: 4px;
  letter-spacing: 0.04em;
}
.directive-icon {
  font-size: 12px;
  animation: pulse-soft 1.4s ease-in-out infinite;
}

/* 输入区 */
.chat-input-area {
  display: flex;
  gap: 10px;
  padding: 14px 16px;
  border-top: 1px solid var(--border);
  background: rgba(5,8,19,0.6);
}
.chat-input {
  flex: 1;
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  padding: 10px 12px;
  font-size: 13px;
  resize: none;
  font-family: var(--font-body);
  color: var(--text-primary);
  background: rgba(255,255,255,0.04);
  outline: none;
  transition: var(--transition);
  line-height: 1.5;
}
.chat-input::placeholder { color: var(--text-muted); }
.chat-input:focus {
  border-color: transparent;
  box-shadow: 0 0 0 1.5px var(--green), 0 0 20px rgba(16,219,161,0.3);
  background: rgba(255,255,255,0.06);
}
.send-btn {
  padding: 0 18px;
  border-radius: var(--radius-sm);
  background: var(--gradient-primary);
  background-size: 200% auto;
  color: white;
  border: none;
  font-family: var(--font-display);
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  transition: var(--transition);
  white-space: nowrap;
  letter-spacing: 0.04em;
  box-shadow: 0 4px 16px rgba(16,219,161,0.3);
}
.send-btn:hover:not(:disabled) {
  background-position: right center;
  transform: translateY(-2px);
  box-shadow: 0 0 30px rgba(16,219,161,0.5), 0 0 60px rgba(255,77,141,0.3);
}
.send-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  box-shadow: none;
}

/* 面板动画 */
.panel-slide-enter-active,
.panel-slide-leave-active {
  transition: opacity 0.3s ease, transform 0.4s var(--ease-elastic);
}
.panel-slide-enter-from,
.panel-slide-leave-to {
  opacity: 0;
  transform: translateY(30px) scale(0.92);
}

@media (max-width: 480px) {
  .assistant-panel {
    right: 12px;
    left: 12px;
    bottom: 90px;
    width: auto;
    max-height: 72vh;
  }
  .assistant-fab {
    bottom: 20px;
    right: 20px;
  }
}
</style>
