<script setup>
import { ref, onMounted, nextTick } from 'vue'
import chatIcon from './components/chatIcon.vue'

class ChatBot {
  constructor() {
    this.rules = [
      {
        keywords: ['سلام', 'درود', 'وقت بخیر', 'hi', 'hello'],
        response: 'سلام! 😊 خوش اومدید. چطور می‌تونم در خرید یا خدمات به شما کمک کنم؟',
      },
      {
        keywords: ['خداحافظ', 'بای', 'ممنون', 'متشکرم', 'مرسی'],
        response: 'خوشحالم که تونستم کمک کنم. امیدوارم تجربه خرید خوبی داشته باشید! 👋',
      },
      {
        keywords: ['ربات', 'انسان', 'ادمین', 'اپراتور'],
        response:
          'من هوش مصنوعی هستم 🤖، اما تمام تلاشم رو می‌کنم مشکلت رو حل کنم. اگر نتونستم، به پشتیبانی انسانی وصل می‌شوی.',
      },

      // --- بخش 2: پرداخت و مالی (با جزئیات دقیق) ---
      {
        keywords: ['روش پرداخت', 'چطور پرداخت کنم', 'کارت به کارت', 'نحوه واریز'],
        response:
          'شما می‌توانید از طریق درگاه امن بانکی (کلیه کارت‌های عضو شتاب) یا کیف پول حساب کاربری پرداخت خود را انجام دهید.',
      },
      {
        keywords: ['مشکل پرداخت', 'پرداخت نشد', 'تراکنش ناموفق', 'خطای درگاه', 'ارور بانک'],
        response:
          'اگر پرداخت ناموفق بود ولی پول کم شد، نگران نباشید! مبلغ طی ۷۲ ساعت توسط بانک برگشت می‌خورد. اگر فیلترشکن روشن است، آن را خاموش کرده و مجدد تلاش کنید.',
      },
      {
        keywords: ['پرداخت در محل', 'درب منزل', 'p.o.d'],
        response:
          'پرداخت در محل فقط برای سفارش‌های شهر تهران فعال است. برای سایر شهرها پرداخت باید آنلاین انجام شود.',
      },
      {
        keywords: ['قسطی', 'اقساط', 'چکی'],
        response:
          'در حال حاضر امکان خرید اقساطی یا چکی وجود ندارد، اما می‌توانید از سرویس‌های "الان بخر بعدا پرداخت کن" (مثل اسنپ‌پی) در مرحله تسویه استفاده کنید.',
      },

      // --- بخش 3: سفارش و پیگیری ---
      {
        keywords: ['پیگیری', 'کجاست', 'وضیعت سفارش', 'نیومده'],
        response:
          'برای پیگیری، لطفاً وارد حساب کاربری بخش "سفارش‌های من" شوید یا کد سفارش را همینجا بنویسید تا بررسی کنم.',
      },
      {
        keywords: ['لغو سفارش', 'کنسل کردن', 'نمیخوام'],
        response:
          'اگر سفارش هنوز "پردازش" یا "ارسال" نشده باشد، می‌توانید از بخش پروفایل آن را لغو کنید. در غیر این صورت با پشتیبانی تماس بگیرید.',
      },
      {
        keywords: ['تغییر سفارش', 'ویرایش', 'اضافه کردن', 'کم کردن'],
        response:
          'متاسفانه پس از ثبت نهایی، امکان تغییر اقلام سفارش وجود ندارد. باید سفارش را لغو و مجدد ثبت کنید.',
      },
      {
        keywords: ['فاکتور', 'رسید'],
        response:
          'فاکتور خرید شما به صورت خودکار به ایمیلتان ارسال شده و در جزئیات سفارش در پروفایل هم قابل دانلود است.',
      },

      // --- بخش 4: ارسال و تحویل ---
      {
        keywords: ['هزینه ارسال', 'پول پست', 'هزینه پیک'],
        response:
          'هزینه ارسال برای سفارش‌های بالای ۵۰۰ هزار تومان رایگان است 🚚. برای مبالغ کمتر، هزینه ثابت ۴۵ هزار تومان محاسبه می‌شود.',
      },
      {
        keywords: ['زمان ارسال', 'کی میرسه', 'چقدر طول میکشه'],
        response:
          'سفارشات تهران ۱ روز کاری و شهرستان‌ها بین ۲ تا ۴ روز کاری با پست پیشتاز تحویل داده می‌شوند.',
      },
      {
        keywords: ['تغییر آدرس', 'آدرس اشتباه'],
        response:
          'اگر سفارش هنوز ارسال نشده، سریعاً با پشتیبانی تماس بگیرید تا آدرس را اصلاح کنند. پس از ارسال، امکان تغییر آدرس نیست.',
      },

      // --- بخش 5: حساب کاربری ---
      {
        keywords: ['رمز عبور', 'پسورد', 'فراموشی', 'وارد نمیشه'],
        response:
          'اگر رمز عبور را فراموش کرده‌اید، در صفحه ورود روی گزینه "رمز عبور را فراموش کرده‌ام" کلیک کنید تا لینک بازیابی پیامک شود.',
      },
      {
        keywords: ['ثبت نام', 'عضویت', 'ساخت اکانت'],
        response:
          'برای ثبت نام کافیست شماره موبایل خود را در بخش "ورود/ثبت نام" (گوشه بالا سمت چپ) وارد کرده و کد تایید را بزنید.',
      },

      // --- بخش 6: محصولات و موجودی ---
      {
        keywords: ['موجودی', 'تموم شده', 'ناموجود'],
        response:
          'اگر کالایی ناموجود است، می‌توانید دکمه "موجود شد خبرم کن" را بزنید تا به محض شارژ انبار به شما پیامک دهیم.',
      },
      {
        keywords: ['سایز', 'اندازه', 'راهنمای سایز'],
        response:
          'در صفحه هر محصول پوشیدنی، یک جدول "راهنمای سایز" وجود دارد. لطفاً قبل از خرید حتماً آن را چک کنید.',
      },
      {
        keywords: ['اصل', 'اورجینال', 'فیک', 'تقلبی'],
        response:
          'تمامی محصولات ما با ضمانت اصالت کالا فروخته می‌شوند و در صورت عدم تطابق، وجه شما کامل عودت داده می‌شود.',
      },

      // --- بخش 7: بازگشت و گارانتی ---
      {
        keywords: ['مرجوع', 'پس دادن', 'بازگشت کالا', 'خراب بود'],
        response:
          'شما ۷ روز ضمانت بازگشت دارید. اگر کالا ایراد دارد یا با عکس مغایرت دارد، تیکت بزنید یا با پشتیبانی تماس بگیرید.',
      },
      {
        keywords: ['شرایط مرجوعی'],
        response:
          'برای مرجوعی، کالا نباید استفاده شده باشد و باید جعبه و ملزومات آن سالم بماند (مگر در موارد خرابی فنی).',
      },
    ]
  }

  getResponse(message) {
    const text = message.toLowerCase()
    const matched = this.rules.filter((rule) => rule.keywords.some((k) => text.includes(k)))
    if (matched.length === 0)
      return 'متوجه منظورت نشدم 😅 لطفاً دقیق‌تر بپرس یا از کلیدواژه‌های مثل خرید، ارسال یا پشتیبانی استفاده کن.'
    return matched.map((m) => m.response).join(' ')
  }
}

const isOpen = ref(false)
const userInput = ref('')
const messages = ref([])
const isTyping = ref(false)
const chatBody = ref(null)
const bot = new ChatBot()

const toggleChat = () => (isOpen.value = !isOpen.value)

const send = async () => {
  if (!userInput.value.trim()) return
  messages.value.push({ text: userInput.value, isBot: false })
  const input = userInput.value
  userInput.value = ''

  await nextTick(() =>
    chatBody.value.scrollTo({
      top: chatBody.value.scrollHeight,
      behavior: 'smooth',
    }),
  )

  isTyping.value = true
  setTimeout(async () => {
    const reply = bot.getResponse(input)
    isTyping.value = false
    messages.value.push({ text: reply, isBot: true })
    await nextTick(() =>
      chatBody.value.scrollTo({
        top: chatBody.value.scrollHeight,
        behavior: 'smooth',
      }),
    )
  }, 1000)
}

onMounted(() => {
  messages.value.push({
    text: 'سلام! 👋 من پشتیبان فروشگاه هستم، چطور کمکتون کنم؟',
    isBot: true,
  })
})
</script>

<template>
  <div class="chat-widget-container" style="direction: rtl">
    <transition name="fade">
      <div v-if="isOpen" class="chat-window">
        <div class="chat-header">
          <h2 class="chat-title">پشتیبان فروشگاه</h2>
          <button @click="toggleChat" class="chat-close-btn">✕</button>
        </div>

        <div ref="chatBody" class="chat-messages-body">
          <div
            v-for="(msg, index) in messages"
            :key="index"
            :class="msg.isBot ? 'message-wrapper-bot' : 'message-wrapper-user'"
          >
            <div :class="msg.isBot ? 'message-bubble message-bot' : 'message-bubble message-user'">
              {{ msg.text }}
            </div>
          </div>

          <div v-if="isTyping" class="typing-indicator-wrapper">
            <div class="dot"></div>
            <div class="dot"></div>
            <div class="dot"></div>
          </div>
        </div>

        <div class="chat-input-area">
          <input
            v-model="userInput"
            @keyup.enter="send"
            type="text"
            placeholder="پیام خود را بنویسید..."
            class="chat-input"
          />
          <button @click="send" class="chat-send-btn">ارسال</button>
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
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    Oxygen,
    Ubuntu,
    Cantarell,
    'Open Sans',
    'Helvetica Neue',
    sans-serif;
  position: fixed;
  bottom: 1.5rem;
  right: 1.5rem;
  z-index: 50;
  border: none;
}
.chat-window {
  min-height: 400px;
  background-color: white;
  box-shadow:
    0 20px 25px -5px rgba(0, 0, 0, 0.1),
    0 8px 10px -6px rgba(0, 0, 0, 0.1);
  border-radius: 1rem;
  padding: 1rem;
  width: 20rem;
  max-height: 70vh;
  display: flex;
  flex-direction: column;
}
.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
  border-bottom: 1px solid #e5e7eb;
  padding-bottom: 0.5rem;
}
.chat-title {
  font-size: 1.125rem;
  font-weight: 600;
  color: #4b5563;
}
.chat-close-btn {
  color: #6b7280;
  transition: all 0.15s ease-in-out;
  border: none;
  background: none;
  cursor: pointer;
}
.chat-close-btn:hover {
  color: #ef4444;
}
.chat-messages-body {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 0.75rem;
}
.chat-messages-body > div:not(:last-child) {
  margin-bottom: 0.75rem;
}
.message-wrapper-bot {
  text-align: left;
}
.message-wrapper-user {
  text-align: right;
}
.message-bubble {
  display: inline-block;
  padding: 0.5rem 0.75rem;
  border-radius: 0.75rem;
  font-size: 0.875rem;
  max-width: 80%;
}
.message-bot {
  background-color: #f3f4f6;
  color: #1f2937;
}
.message-user {
  background-color: #3b82f6;
  color: white;
}
.chat-input-area {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
.chat-input {
  flex: 1;
  border: 1px solid #d1d5db;
  border-radius: 0.75rem;
  padding: 0.5rem 0.75rem;
  font-size: 0.875rem;
  outline: none;
  transition: box-shadow 0.15s;
  font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    Oxygen,
    Ubuntu,
    Cantarell,
    'Open Sans',
    'Helvetica Neue',
    sans-serif;
}
.chat-input:focus {
  border-color: #60a5fa;
  box-shadow: 0 0 0 3px rgba(96, 165, 250, 0.5);
}
.chat-send-btn {
  background-color: #3b82f6;
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 0.75rem;
  transition: background-color 0.15s;
  border: none;
  cursor: pointer;
  font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    Oxygen,
    Ubuntu,
    Cantarell,
    'Open Sans',
    'Helvetica Neue',
    sans-serif;
}
.chat-send-btn:hover {
  background-color: #2563eb;
}
.chat-toggle-btn {
  background-color: #2563eb;
  color: white;
  padding: 1rem;
  border-radius: 9999px;
  box-shadow:
    0 10px 15px -3px rgba(0, 0, 0, 0.1),
    0 4px 6px -4px rgba(0, 0, 0, 0.05);
  transition: transform 0.2s ease-in-out;
  border: none;
  cursor: pointer;
}
.chat-toggle-btn:hover {
  transform: scale(1.05);
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(5px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
.message-bubble {
  animation: slideIn 0.25s ease;
}

.dot {
  width: 6px;
  height: 6px;
  background-color: #ccc;
  border-radius: 50%;
  animation: blink 1.2s infinite;
}
.dot:nth-child(2) {
  animation-delay: 0.2s;
}
.dot:nth-child(3) {
  animation-delay: 0.4s;
}
@keyframes blink {
  0%,
  80%,
  100% {
    opacity: 0;
  }
  40% {
    opacity: 1;
  }
}
</style>
