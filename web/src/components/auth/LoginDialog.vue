<template>
  <Teleport to="body">
    <Transition name="fade">
      <div v-if="visible" class="dialog-mask" @click.self="close">
        <div class="dialog-box">
          <button class="close-btn" @click="close">×</button>
          <div class="dialog-brand">
            <svg viewBox="0 0 48 48" width="40" height="40" fill="none">
              <defs>
                <linearGradient id="loginGrad" x1="0" y1="0" x2="48" y2="48" gradientUnits="userSpaceOnUse">
                  <stop offset="0%" stop-color="#6366f1"/>
                  <stop offset="100%" stop-color="#818cf8"/>
                </linearGradient>
              </defs>
              <rect width="48" height="48" rx="12" fill="url(#loginGrad)"/>
              <text x="24" y="33" text-anchor="middle" fill="white" font-size="22" font-weight="bold">V</text>
            </svg>
          </div>
          <div class="dialog-header">
            <button
              class="tab-btn"
              :class="{ active: mode === 'login' }"
              @click="mode = 'login'"
            >
              登录
            </button>
            <button
              class="tab-btn"
              :class="{ active: mode === 'register' }"
              @click="mode = 'register'"
            >
              注册
            </button>
          </div>

          <form class="dialog-form" @submit.prevent="handleSubmit">
            <div class="form-item">
              <div class="input-icon">
                <svg viewBox="0 0 24 24" width="18" height="18" fill="currentColor"><path d="M20 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z"/></svg>
              </div>
              <input v-model="form.email" type="email" placeholder="邮箱" required />
            </div>
            <div class="form-item">
              <div class="input-icon">
                <svg viewBox="0 0 24 24" width="18" height="18" fill="currentColor"><path d="M18 8h-1V6c0-2.76-2.24-5-5-5S7 3.24 7 6v2H6c-1.1 0-2 .9-2 2v10c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2V10c0-1.1-.9-2-2-2zm-6 9c-1.1 0-2-.9-2-2s.9-2 2-2 2 .9 2 2-.9 2-2 2zM9 8V6c0-1.66 1.34-3 3-3s3 1.34 3 3v2H9z"/></svg>
              </div>
              <input
                v-model="form.password"
                type="password"
                :placeholder="mode === 'login' ? '密码' : '设置密码'"
                required
              />
            </div>
            <div v-if="mode === 'register'" class="form-item">
              <div class="input-icon">
                <svg viewBox="0 0 24 24" width="18" height="18" fill="currentColor"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/></svg>
              </div>
              <input v-model="form.nickName" type="text" placeholder="昵称" required />
            </div>
            <div class="form-item captcha-row">
              <div class="input-icon">
                <svg viewBox="0 0 24 24" width="18" height="18" fill="currentColor"><path d="M12 1L3 14h7l-2 9 10-13h-7l2-9z"/></svg>
              </div>
              <input v-model="form.checkCode" type="text" placeholder="验证码" required />
              <img
                v-if="captchaImg"
                :src="captchaImg"
                class="captcha-img"
                alt="验证码"
                @click="refreshCaptcha"
              />
              <button v-else type="button" class="captcha-placeholder" @click="refreshCaptcha">
                获取验证码
              </button>
            </div>
            <p v-if="errorMsg" class="error-msg">{{ errorMsg }}</p>
            <button type="submit" class="btn-primary submit-btn" :disabled="loading">
              {{ loading ? '提交中...' : mode === 'login' ? '登 录' : '注 册' }}
            </button>
          </form>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { ref, watch, reactive } from 'vue'
import { useUserStore } from '@/stores'
import { accountApi } from '@/api'
import { parseCaptchaResponse } from '@/utils/captcha'
import { useRouter, useRoute } from 'vue-router'

const props = defineProps({
  visible: Boolean
})

const emit = defineEmits(['update:visible'])

const userStore = useUserStore()
const router = useRouter()
const route = useRoute()
const mode = ref('login')
const loading = ref(false)
const errorMsg = ref('')
const captchaImg = ref('')
const checkCodeKey = ref('')

const form = reactive({
  email: '',
  password: '',
  nickName: '',
  checkCode: ''
})

function close() {
  emit('update:visible', false)
  errorMsg.value = ''
}

async function refreshCaptcha() {
  try {
    const res = await accountApi.checkCode()
    const { img, key } = parseCaptchaResponse(res)
    captchaImg.value = img
    checkCodeKey.value = key
    errorMsg.value = ''
  } catch (e) {
    captchaImg.value = ''
    checkCodeKey.value = ''
    errorMsg.value = e.message || '获取验证码失败'
  }
}

async function handleSubmit() {
  loading.value = true
  errorMsg.value = ''
  try {
    const data = new FormData()
    data.append('email', form.email)
    data.append('checkCodeKey', checkCodeKey.value)
    data.append('checkCode', form.checkCode)

    if (mode.value === 'login') {
      data.append('password', form.password)
      await userStore.login(data)
      close()
      const redirect = route.query.redirect || '/account/home'
      router.push(String(redirect))
    } else {
      data.append('registerPassword', form.password)
      data.append('nickName', form.nickName)
      await accountApi.register(data)
      mode.value = 'login'
      errorMsg.value = '注册成功，请登录'
      refreshCaptcha()
    }
  } catch (e) {
    errorMsg.value = e.message || '操作失败'
    refreshCaptcha()
  } finally {
    loading.value = false
  }
}

watch(
  () => props.visible,
  (val) => {
    if (val) {
      refreshCaptcha()
      form.email = ''
      form.password = ''
      form.nickName = ''
      form.checkCode = ''
    }
  }
)
</script>

<style scoped lang="scss">
.dialog-mask {
  position: fixed;
  inset: 0;
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(8px);
}

.dialog-box {
  position: relative;
  width: 420px;
  padding: 36px;
  background: var(--bili-white);
  border-radius: var(--bili-radius-xl);
  box-shadow: var(--bili-shadow-lg);
  animation: fadeInUp 0.3s ease;
}

.dialog-brand {
  display: flex;
  justify-content: center;
  margin-bottom: 24px;
}

.close-btn {
  position: absolute;
  top: 14px;
  right: 18px;
  font-size: 22px;
  color: var(--bili-text-tertiary);
  line-height: 1;
  transition: color var(--bili-transition-fast);

  &:hover {
    color: var(--bili-text);
  }
}

.dialog-header {
  display: flex;
  gap: 32px;
  margin-bottom: 28px;
  justify-content: center;
}

.tab-btn {
  font-size: 18px;
  font-weight: 600;
  color: var(--bili-text-tertiary);
  padding-bottom: 10px;
  border-bottom: 2.5px solid transparent;
  transition: all var(--bili-transition);

  &.active {
    color: var(--bili-pink);
    border-bottom-color: var(--bili-pink);
  }

  &:hover:not(.active) {
    color: var(--bili-text-secondary);
  }
}

.form-item {
  margin-bottom: 16px;
  display: flex;
  align-items: center;
  gap: 0;
  position: relative;

  .input-icon {
    position: absolute;
    left: 14px;
    top: 50%;
    transform: translateY(-50%);
    color: var(--bili-text-tertiary);
    z-index: 1;
    display: flex;
    align-items: center;
  }

  input {
    width: 100%;
    height: 46px;
    padding: 0 16px 0 42px;
    border: 1.5px solid var(--bili-border);
    border-radius: var(--bili-radius);
    font-size: 14px;
    transition: all var(--bili-transition);

    &:focus {
      border-color: var(--bili-pink);
      box-shadow: 0 0 0 3px var(--bili-pink-light);
    }

    &::placeholder {
      color: var(--bili-text-tertiary);
    }
  }
}

.captcha-row {
  display: flex;
  gap: 12px;

  input {
    flex: 1;
  }
}

.captcha-img {
  width: 120px;
  height: 46px;
  border-radius: var(--bili-radius);
  cursor: pointer;
  object-fit: cover;
  border: 1.5px solid var(--bili-border);
  transition: border-color var(--bili-transition);

  &:hover {
    border-color: var(--bili-pink);
  }
}

.captcha-placeholder {
  width: 120px;
  height: 46px;
  border-radius: var(--bili-radius);
  border: 1.5px dashed var(--bili-border);
  font-size: 12px;
  color: var(--bili-text-tertiary);
  flex-shrink: 0;
  transition: all var(--bili-transition);

  &:hover {
    border-color: var(--bili-pink);
    color: var(--bili-pink);
  }
}

.error-msg {
  color: var(--bili-danger);
  font-size: 13px;
  margin-bottom: 12px;
  padding: 8px 12px;
  background: rgba(245, 63, 63, 0.06);
  border-radius: var(--bili-radius);
}

.submit-btn {
  width: 100%;
  margin-top: 8px;
  height: 46px;
  font-size: 16px;
  font-weight: 600;
  border-radius: var(--bili-radius);
}
</style>
