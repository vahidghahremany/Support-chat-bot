<script setup>
import { ref, onMounted, nextTick, watch } from 'vue'
import chatIcon from './components/chatIcon.vue'
import userIcon from './components/userIcon.vue'
import closeIcon from './components/closeIcon.vue'
import clearIcon from './components/clearIcon.vue'
import sendIcon from './components/sendIcon.vue'

class ChatBot {
  constructor() {
    this.rules = [
      {
        keywords: ['سلام', 'درود', 'وقت بخیر', 'hi', 'hello'],
        response: 'سلام! 😊 خوش اومدید. چطور می‌تونم در خرید یا خدمات به شما کمک کنم؟',
        score: 0,
      },
      {
        keywords: ['خداحافظ', 'بای', 'ممنون', 'متشکرم', 'مرسی'],
        response: 'خوشحالم که تونستم کمک کنم. امیدوارم تجربه خرید خوبی داشته باشید! 👋',
        score: 0,
      },
      {
        keywords: ['ربات', 'انسان', 'ادمین', 'اپراتور'],
        response:
          'من هوش مصنوعی هستم 🤖، اما تمام تلاشم رو می‌کنم مشکلت رو حل کنم. اگر نتونستم، به پشتیبانی انسانی وصل می‌شوی.',
        score: 1,
      },
      {
        keywords: ['پشتیبانی', 'پشتیبان', 'تماس با شما', 'اپراتور انسانی'],
        response:
          'برای ارتباط با پشتیبانی انسانی، می‌توانید در ساعات اداری (۹ صبح تا ۵ عصر) با شماره ۰۲۱-۱۲۳۴۵۶ تماس بگیرید یا از طریق صفحه «تماس با ما» تیکت ثبت کنید.',
        score: 3,
      },
      {
        keywords: ['روش پرداخت', 'چطور پرداخت کنم', 'کارت به کارت', 'نحوه واریز'],
        response:
          'شما می‌توانید از طریق درگاه امن بانکی (کلیه کارت‌های عضو شتاب) یا کیف پول حساب کاربری پرداخت خود را انجام دهید.',
        score: 3,
      },
      {
        keywords: ['مشکل پرداخت', 'پرداخت نشد', 'تراکنش ناموفق', 'خطای درگاه', 'ارور بانک'],
        response:
          'اگر پرداخت ناموفق بود ولی پول کم شد، نگران نباشید! مبلغ طی ۷۲ ساعت توسط بانک برگشت می‌خورد. اگر فیلترشکن روشن است، آن را خاموش کرده و مجدد تلاش کنید.',
        score: 4,
      },
      {
        keywords: ['پرداخت در محل', 'درب منزل', 'p.o.d'],
        response:
          'پرداخت در محل فقط برای سفارش‌های شهر تهران فعال است. برای سایر شهرها پرداخت باید آنلاین انجام شود.',
        score: 3,
      },
      {
        keywords: ['قسطی', 'اقساط', 'چکی'],
        response:
          'در حال حاضر امکان خرید اقساطی یا چکی وجود ندارد، اما می‌توانید از سرویس‌های "الان بخر بعدا پرداخت کن" (مثل اسنپ‌پی) در مرحله تسویه استفاده کنید.',
        score: 3,
      },
      {
        keywords: ['پیگیری', 'کجاست', 'وضیعت سفارش', 'نیومده'],
        response:
          'برای پیگیری، لطفاً وارد حساب کاربری بخش "سفارش‌های من" شوید یا کد سفارش را همینجا بنویسید تا بررسی کنم.',
        score: 5,
      },
      {
        keywords: ['لغو سفارش', 'کنسل', 'نمیخوام'],
        response:
          'اگر سفارش هنوز "پردازش" یا "ارسال" نشده باشد، می‌توانید از بخش پروفایل آن را لغو کنید. در غیر این صورت با پشتیبانی تماس بگیرید.',
        score: 4,
      },
      {
        keywords: ['تغییر سفارش', 'ویرایش', 'اضافه', 'کم کردن'],
        response:
          'متاسفانه پس از ثبت نهایی، امکان تغییر اقلام سفارش وجود ندارد. باید سفارش را لغو و مجدد ثبت کنید.',
        score: 3,
      },
      {
        keywords: ['فاکتور', 'رسید'],
        response:
          'فاکتور خرید شما به صورت خودکار به ایمیلتان ارسال شده و در جزئیات سفارش در پروفایل هم قابل دانلود است.',
        score: 2,
      },
      {
        keywords: ['هزینه ارسال', 'پول پست', 'هزینه پیک'],
        response:
          'هزینه ارسال برای سفارش‌های بالای ۵۰۰ هزار تومان رایگان است 🚚. برای مبالغ کمتر، هزینه ثابت ۴۵ هزار تومان محاسبه می‌شود.',
        score: 3,
      },
      {
        keywords: ['زمان ارسال', 'کی میرسه', 'چقدر طول میکشه'],
        response:
          'سفارشات تهران ۱ روز کاری و شهرستان‌ها بین ۲ تا ۴ روز کاری با پست پیشتاز تحویل داده می‌شوند.',
        score: 3,
      },
      {
        keywords: ['تغییر آدرس', 'آدرس اشتباه'],
        response:
          'اگر سفارش هنوز ارسال نشده، سریعاً با پشتیبانی تماس بگیرید تا آدرس را اصلاح کنند. پس از ارسال، امکان تغییر آدرس نیست.',
        score: 4,
      },
      {
        keywords: ['رمز عبور', 'پسورد', 'فراموشی', 'وارد نمیشه'],
        response:
          'اگر رمز عبور را فراموش کرده‌اید، در صفحه ورود روی گزینه "رمز عبور را فراموش کرده‌ام" کلیک کنید تا لینک بازیابی پیامک شود.',
        score: 2,
      },
      {
        keywords: ['ثبت نام', 'عضویت', 'ساخت اکانت'],
        response:
          'برای ثبت نام کافیست شماره موبایل خود را در بخش "ورود/ثبت نام" (گوشه بالا سمت چپ) وارد کرده و کد تایید را بزنید.',
        score: 2,
      },
      {
        keywords: ['موجودی', 'تموم شده', 'ناموجود'],
        response:
          'اگر کالایی ناموجود است، می‌توانید دکمه "موجود شد خبرم کن" را بزنید تا به محض شارژ انبار به شما پیامک دهیم.',
        score: 3,
      },
      {
        keywords: ['سایز', 'اندازه', 'راهنمای سایز'],
        response:
          'در صفحه هر محصول پوشیدنی، یک جدول "راهنمای سایز" وجود دارد. لطفاً قبل از خرید حتماً آن را چک کنید.',
        score: 2,
      },
      {
        keywords: ['اصل', 'اورجینال', 'فیک', 'تقلبی'],
        response:
          'تمامی محصولات ما با ضمانت اصالت کالا فروخته می‌شوند و در صورت عدم تطابق، وجه شما کامل عودت داده می‌شود.',
        score: 3,
      },
      {
        keywords: ['مرجوع', 'پس دادن', 'بازگشت کالا', 'خراب بود'],
        response:
          'شما ۷ روز ضمانت بازگشت دارید. اگر کالا ایراد دارد یا با عکس مغایرت دارد، تیکت بزنید یا با پشتیبانی تماس بگیرید.',
        score: 4,
      },
      {
        keywords: ['شرایط مرجوعی'],
        response:
          'برای مرجوعی، کالا نباید استفاده شده باشد و باید جعبه و ملزومات آن سالم بماند (مگر در موارد خرابی فنی).',
        score: 3,
      },
    ]
  }

  getResponse(message) {
    const text = message.toLowerCase()
    let matches = []

    for (const rule of this.rules) {
      let currentScore = 0

      let matchBonus = 0
      for (const keyword of rule.keywords) {
        if (text.includes(keyword)) {
          matchBonus += 1
        }
      }

      if (matchBonus > 0) {
        currentScore = (rule.score || 0) + matchBonus
        matches.push({ response: rule.response, score: currentScore, baseScore: rule.score || 0 })
      }
    }

    if (matches.length === 0) {
      return 'متوجه منظورت نشدم 😅 لطفاً دقیق‌تر بپرس یا از کلیدواژه‌های مثل خرید، ارسال یا پشتیبانی استفاده کن.'
    }

    const maxScore = Math.max(...matches.map((m) => m.score))

    let finalMatches

    if (maxScore <= 1) {
      finalMatches = matches.filter((m) => m.score === maxScore)
    } else {
      finalMatches = matches.filter((m) => m.baseScore > 0)
    }

    const uniqueResponses = [...new Set(finalMatches.map((m) => m.response))]

    if (uniqueResponses.length === 0) {
      if (matches.length > 0) {
        matches.sort((a, b) => b.score - a.score)
        return matches[0].response
      }
      return 'متوجه منظورت نشدم 😅...'
    }

    return uniqueResponses.join('\n\n')
  }
}

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
