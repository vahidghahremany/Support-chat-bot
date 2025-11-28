<script setup>
import { ref, onMounted, nextTick, watch } from 'vue'
import ChatBot from './data/chatBot'
import chatIcon from './components/chatIcon.vue'
import userIcon from './components/userIcon.vue'
import closeIcon from './components/closeIcon.vue'
import clearIcon from './components/clearIcon.vue'
import sendIcon from './components/sendIcon.vue'

const isOpen = ref(false)
const userInput = ref('')
const messages = ref([])
const isTyping = ref(false)
const chatBody = ref(null)
const bot = new ChatBot()

const CHAT_HISTORY_KEY = 'vue-chatbot-history'

onMounted(() => {
  const savedMessages = localStorage.getItem(CHAT_HISTORY_KEY)
  if (savedMessages) {
    messages.value = JSON.parse(savedMessages)
  } else {
    messages.value.push({
      id: crypto.randomUUID(),
      text: 'سلام! 👋 من پشتیبان فروشگاه هستم، چطور کمکتون کنم؟',
      isBot: true,
      timestamp: new Date().toLocaleTimeString('fa-IR', { hour: '2-digit', minute: '2-digit' }),
    })
  }
  scrollToBottom()
})

watch(
  messages,
  (newMessages) => {
    localStorage.setItem(CHAT_HISTORY_KEY, JSON.stringify(newMessages))
  },
  { deep: true },
)

const scrollToBottom = async () => {
  await nextTick()

  setTimeout(() => {
    if (chatBody.value && chatBody.value.$el) {
      const el = chatBody.value.$el
      el.scrollTo({
        top: el.scrollHeight,
        behavior: 'smooth',
      })
    }
  }, 100)
}

const send = async () => {
  if (!userInput.value.trim()) return

  const input = userInput.value.trim()

  messages.value.push({
    id: crypto.randomUUID(),
    text: input,
    isBot: false,
    timestamp: new Date().toLocaleTimeString('fa-IR', { hour: '2-digit', minute: '2-digit' }),
  })

  userInput.value = ''
  await scrollToBottom()

  isTyping.value = true

  setTimeout(
    async () => {
      const replyText = bot.getResponse(input)
      isTyping.value = false

      messages.value.push({
        id: crypto.randomUUID(),
        text: replyText,
        isBot: true,
        timestamp: new Date().toLocaleTimeString('fa-IR', { hour: '2-digit', minute: '2-digit' }),
      })

      await scrollToBottom()
    },
    1200 + Math.random() * 500,
  )
}

const toggleChat = () => {
  isOpen.value = !isOpen.value
  if (isOpen.value) {
    scrollToBottom()
  }
}

const clearChat = () => {
  messages.value = [
    {
      id: crypto.randomUUID(),
      text: 'سلام! 👋 چت شما بازنشانی شد. چطور کمکتون کنم؟',
      isBot: true,
      timestamp: new Date().toLocaleTimeString('fa-IR', { hour: '2-digit', minute: '2-digit' }),
    },
  ]
  localStorage.removeItem(CHAT_HISTORY_KEY)
  scrollToBottom()
}
</script>

<template>
  <div class="chat-widget-container chat-widget-rtl">
    <transition name="window-pop">
      <div v-if="isOpen" class="chat-window">
        <div class="chat-header">
          <div class="chat-header-info">
            <div class="chat-header-avatar">
              <userIcon />
            </div>
            <h2 class="chat-title">پشتیبان فروشگاه</h2>
          </div>
          <div class="chat-header-actions">
            <button @click="clearChat" class="chat-action-btn" title="پاک کردن گفتگو">
              <clearIcon />
            </button>
            <button @click="toggleChat" class="chat-action-btn chat-close-btn" title="بستن">
              <closeIcon />
            </button>
          </div>
        </div>
        <transition-group name="message-fade" tag="div" ref="chatBody" class="chat-messages-body">
          <div
            v-for="msg in messages"
            :key="msg.id"
            :class="msg.isBot ? 'message-wrapper-bot' : 'message-wrapper-user'"
          >
            <div :class="msg.isBot ? 'message-bubble message-bot' : 'message-bubble message-user'">
              {{ msg.text }}
              <span class="message-timestamp">{{ msg.timestamp }}</span>
            </div>
          </div>
          <div v-if="isTyping" key="typing-indicator" class="typing-indicator-wrapper">
            <div class="message-bubble message-bot">
              <div class="dot-flashing"></div>
            </div>
          </div>
        </transition-group>
        <div class="chat-input-area">
          <input
            v-model="userInput"
            @keyup.enter="send"
            type="text"
            placeholder="پیام خود را بنویسید..."
            class="chat-input"
          />
          <button @click="send" class="chat-send-btn" title="ارسال">
            <sendIcon />
          </button>
        </div>
      </div>
    </transition>

    <button v-if="!isOpen" @click="toggleChat" class="chat-toggle-btn">
      <chatIcon />
    </button>
  </div>
</template>

<style scoped>
.chat-widget-container {
  font-family:
    -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, 'Noto Sans',
    sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  position: fixed;
  bottom: 1.5rem;
  right: 1.5rem;
  z-index: 1000;
  border: none;
}
.chat-widget-rtl {
  direction: rtl;
}
.chat-widget-rtl .chat-window {
  text-align: right;
}
.chat-window {
  width: 370px;
  max-height: 70vh;
  min-height: 450px;
  background-color: #ffffff;
  box-shadow:
    0 10px 30px -5px rgba(0, 0, 0, 0.1),
    0 4px 15px -5px rgba(0, 0, 0, 0.07);
  border-radius: 1rem;
  border: 1px solid #e5e7eb;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 1.25rem;
  background-color: #fafafa;
  border-bottom: 1px solid #e5e7eb;
  flex-shrink: 0;
}
.chat-header-info {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}
.chat-header-avatar {
  width: 2.5rem;
  height: 2.5rem;
  border-radius: 50%;
  background-color: #e5e7eb;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #4b5563;
}
.chat-header-avatar svg {
  width: 1.5rem;
  height: 1.5rem;
}
.chat-title {
  font-size: 1.125rem;
  font-weight: 600;
  color: #1f2937;
}
.chat-header-actions {
  display: flex;
  align-items: center;
  gap: 0.25rem;
}
.chat-action-btn {
  border: none;
  background: none;
  cursor: pointer;
  color: #6b7280;
  padding: 0.25rem;
  border-radius: 9999px;
  transition: all 0.2s ease-in-out;
}
.chat-action-btn svg {
  width: 1.6rem;
  height: 1.6rem;
}
.chat-action-btn:hover {
  color: #1f2937;
  background-color: #e5e7eb;
}
.chat-close-btn:hover {
  color: #ef4444;
  background-color: #fee2e2;
}
.chat-messages-body {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.message-wrapper-bot {
  display: flex;
  justify-content: flex-start;
}
.message-wrapper-user {
  display: flex;
  justify-content: flex-end;
}
.message-bubble {
  display: inline-block;
  padding: 0.625rem 1rem;
  border-radius: 1.125rem;
  font-size: 0.9375rem;
  max-width: 85%;
  line-height: 1.6;
  position: relative;
  white-space: pre-wrap;
}
.message-bot {
  background-color: #f3f4f6;
  color: #1f2937;
  border-bottom-left-radius: 0.25rem;
}
.message-user {
  background-image: linear-gradient(to top right, #3b82f6, #2563eb);
  color: white;
  border-bottom-right-radius: 0.25rem;
}
.message-timestamp {
  display: block;
  font-size: 0.75rem;
  color: #9ca3af;
  margin-top: 0.25rem;
  text-align: left;
}
.message-user .message-timestamp {
  color: rgba(255, 255, 255, 0.7);
}
.chat-input-area {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem 1.25rem;
  border-top: 1px solid #e5e7eb;
  background-color: #f9fafb;
}
.chat-input {
  flex: 1;
  border: 1px solid #d1d5db;
  border-radius: 9999px;
  padding: 0.75rem 1rem;
  font-size: 0.9375rem;
  outline: none;
  transition-property: border-color, box-shadow;
  transition-duration: 0.2s;
  font-family: inherit;
  background-color: #ffffff;
}
.chat-input:focus {
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
}
.chat-send-btn {
  background-color: #3b82f6;
  color: white;
  width: 2.75rem;
  height: 2.75rem;
  border-radius: 50%;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.2s ease-in-out;
  flex-shrink: 0;
}
.chat-send-btn svg {
  width: 1.3rem;
  height: 1.3rem;
  transform: scaleX(1);
}
.chat-widget-rtl .chat-send-btn svg {
  transform: scaleX(-1);
}
.chat-send-btn:hover {
  background-color: #2563eb;
}

.chat-toggle-btn {
  background-color: #2563eb;
  color: white;
  width: 3.75rem;
  height: 3.75rem;
  padding: 1rem;
  border-radius: 9999px;
  box-shadow:
    0 10px 20px -5px rgba(37, 99, 235, 0.3),
    0 4px 10px -4px rgba(0, 0, 0, 0.05);
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  animation: pulse 2s infinite;
}
.chat-toggle-btn svg {
  width: 100%;
  height: 100%;
}
.chat-toggle-btn:hover {
  transform: translateY(-4px) scale(1.05);
  box-shadow:
    0 15px 25px -5px rgba(37, 99, 235, 0.4),
    0 6px 15px -4px rgba(0, 0, 0, 0.07);
  animation: none;
}
@keyframes pulse {
  0%,
  100% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.05);
    opacity: 0.9;
  }
}

.window-pop-enter-active,
.window-pop-leave-active {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
.window-pop-enter-from,
.window-pop-leave-to {
  opacity: 0;
  transform: translateY(20px) scale(0.95);
}
.message-fade-enter-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}
.message-fade-leave-active {
  transition: all 0.3s ease;
}
.message-fade-enter-from {
  opacity: 0;
  transform: translateY(15px);
}
.message-fade-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.typing-indicator-wrapper {
  display: flex;
  justify-content: flex-start;
}
.dot-flashing {
  position: relative;
  width: 8px;
  height: 8px;
  border-radius: 5px;
  background-color: #9ca3af;
  color: #9ca3af;
  animation: dot-flashing 1s infinite linear alternate;
  animation-delay: 0.5s;
}
.dot-flashing::before,
.dot-flashing::after {
  content: '';
  display: inline-block;
  position: absolute;
  top: 0;
}
.dot-flashing::before {
  left: -15px;
  width: 8px;
  height: 8px;
  border-radius: 5px;
  background-color: #9ca3af;
  color: #9ca3af;
  animation: dot-flashing 1s infinite alternate;
  animation-delay: 0s;
}
.dot-flashing::after {
  left: 15px;
  width: 8px;
  height: 8px;
  border-radius: 5px;
  background-color: #9ca3af;
  color: #9ca3af;
  animation: dot-flashing 1s infinite alternate;
  animation-delay: 1s;
}

@keyframes dot-flashing {
  0% {
    background-color: #9ca3af;
  }
  50%,
  100% {
    background-color: #d1d5db;
  }
}
</style>
