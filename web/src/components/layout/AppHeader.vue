<template>
  <header class="app-header" :class="{ scrolled: isScrolled }">
    <div class="header-inner container">
      <router-link to="/" class="logo">
        <svg class="logo-icon" viewBox="0 0 48 48" fill="none">
          <defs>
            <linearGradient id="logoGrad" x1="0" y1="0" x2="48" y2="48" gradientUnits="userSpaceOnUse">
              <stop offset="0%" stop-color="#6366f1"/>
              <stop offset="100%" stop-color="#818cf8"/>
            </linearGradient>
          </defs>
          <rect width="48" height="48" rx="12" fill="url(#logoGrad)"/>
          <text x="24" y="33" text-anchor="middle" fill="white" font-size="22" font-weight="bold">V</text>
        </svg>
        <span class="logo-text">VistaMind</span>
      </router-link>

      <div class="search-box" @click.stop>
        <form class="search-form" @submit.prevent="handleSearch">
          <input
            v-model="keyword"
            type="text"
            placeholder="搜索视频、UP主..."
            class="search-input"
            @focus="onSearchFocus"
          />
          <button type="submit" class="search-btn" aria-label="搜索">
            <svg viewBox="0 0 24 24" width="18" height="18" fill="currentColor">
              <path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/>
            </svg>
          </button>
        </form>
        <div v-if="showSuggest && hotKeywords.length" class="search-suggest">
          <div class="suggest-title">
            <svg viewBox="0 0 24 24" width="14" height="14" fill="var(--bili-pink)" style="margin-right:4px">
              <path d="M13 1L3 14h7l-2 9 10-13h-7l2-9z"/>
            </svg>
            VistaMind 热搜
          </div>
          <div
            v-for="(item, index) in hotKeywords"
            :key="index"
            class="suggest-item"
            @click="searchKeyword(item)"
          >
            <span class="rank" :class="{ top: index < 3 }">{{ index + 1 }}</span>
            <span>{{ item }}</span>
          </div>
        </div>
      </div>

      <nav class="header-nav">
        <router-link v-if="userStore.isLoggedIn" to="/upload" class="nav-item upload-btn">
          <svg viewBox="0 0 24 24" width="18" height="18" fill="currentColor">
            <path d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"/>
          </svg>
          <span>投稿</span>
        </router-link>

        <router-link v-if="userStore.isLoggedIn" to="/account/history" class="nav-item" title="历史">
          <svg viewBox="0 0 24 24" width="22" height="22" fill="currentColor">
            <path d="M13 3a9 9 0 0 0-9 9H1l3.89 3.89.07.14L9 12H6c0-3.87 3.13-7 7-7s7 3.13 7 7-3.13 7-7 7c-1.93 0-3.68-.79-4.94-2.06l-1.42 1.42A8.954 8.954 0 0 0 13 21a9 9 0 0 0 0-18zm-1 5v5l4.28 2.54.72-1.21-3.5-2.08V8H12z"/>
          </svg>
        </router-link>

        <router-link v-if="userStore.isLoggedIn" to="/account/message" class="nav-item" title="消息">
          <svg viewBox="0 0 24 24" width="22" height="22" fill="currentColor">
            <path d="M20 2H4c-1.1 0-2 .9-2 2v18l4-4h14c1.1 0 2-.9 2-2V4c0-1.1-.9-2-2-2zm0 14H6l-2 2V4h16v12z"/>
          </svg>
          <span v-if="userStore.noReadCount" class="badge">{{ userStore.noReadCount > 99 ? '99+' : userStore.noReadCount }}</span>
        </router-link>

        <div v-if="userStore.isLoggedIn" class="user-avatar-wrap">
          <router-link :to="`/user/${userStore.userInfo.userId}`" class="user-avatar">
            <img :src="avatarUrl" :alt="userStore.userInfo.nickName" />
          </router-link>
          <div class="user-dropdown">
            <div class="dropdown-header">
              <img :src="avatarUrl" class="dropdown-avatar" alt="" />
              <span class="dropdown-name">{{ userStore.userInfo.nickName }}</span>
            </div>
            <div class="dropdown-divider"></div>
            <router-link to="/account/home" class="dropdown-item">
              <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/></svg>
              个人中心
            </router-link>
            <router-link to="/account/videos" class="dropdown-item">
              <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><path d="M17 10.5V7c0-.55-.45-1-1-1H4c-.55 0-1 .45-1 1v10c0 .55.45 1 1 1h12c.55 0 1-.45 1-1v-3.5l4 4v-11l-4 4z"/></svg>
              我的投稿
            </router-link>
            <router-link to="/account/collection" class="dropdown-item">
              <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><path d="M17 3H7c-1.1 0-2 .9-2 2v16l7-3 7 3V5c0-1.1-.9-2-2-2z"/></svg>
              我的收藏
            </router-link>
            <router-link to="/account/settings" class="dropdown-item">
              <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><path d="M19.14 12.94c.04-.3.06-.61.06-.94 0-.32-.02-.64-.07-.94l2.03-1.58a.49.49 0 0 0 .12-.61l-1.92-3.32a.49.49 0 0 0-.59-.22l-2.39.96c-.5-.38-1.03-.7-1.62-.94l-.36-2.54a.484.484 0 0 0-.48-.41h-3.84c-.24 0-.43.17-.47.41l-.36 2.54c-.59.24-1.13.57-1.62.94l-2.39-.96c-.22-.08-.47 0-.59.22L2.74 8.87c-.12.21-.08.47.12.61l2.03 1.58c-.05.3-.07.62-.07.94s.02.64.07.94l-2.03 1.58a.49.49 0 0 0-.12.61l1.92 3.32c.12.22.37.29.59.22l2.39-.96c.5.38 1.03.7 1.62.94l.36 2.54c.05.24.24.41.48.41h3.84c.24 0 .44-.17.47-.41l.36-2.54c.59-.24 1.13-.56 1.62-.94l2.39.96c.22.08.47 0 .59-.22l1.92-3.32c.12-.22.07-.47-.12-.61l-2.01-1.58zM12 15.6c-1.98 0-3.6-1.62-3.6-3.6s1.62-3.6 3.6-3.6 3.6 1.62 3.6 3.6-1.62 3.6-3.6 3.6z"/></svg>
              账号设置
            </router-link>
            <div class="dropdown-divider"></div>
            <button class="dropdown-item logout" @click="handleLogout">
              <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><path d="M17 7l-1.41 1.41L18.17 11H8v2h10.17l-2.58 2.58L17 17l5-5zM4 5h8V3H4c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h8v-2H4V5z"/></svg>
              退出登录
            </button>
          </div>
        </div>

        <template v-else>
          <button class="nav-item login-btn" @click="$emit('open-login')">
            <svg viewBox="0 0 24 24" width="22" height="22" fill="currentColor">
              <path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/>
            </svg>
            <span>登录</span>
          </button>
        </template>
      </nav>
    </div>
  </header>
  <div v-if="showSuggest" class="search-mask" @click="showSuggest = false" />
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores'
import { getAvatarUrl } from '@/utils/format'
import { videoApi } from '@/api'

defineEmits(['open-login'])

const FALLBACK_HOT = ['编程', '游戏', '动漫', '音乐', '科技']

const router = useRouter()
const userStore = useUserStore()
const keyword = ref('')
const showSuggest = ref(false)
const hotKeywords = ref([])
const isScrolled = ref(false)
let hotLoading = false

const avatarUrl = computed(() => getAvatarUrl(userStore.userInfo?.avatar))

function normalizeHotList(data) {
  const list = Array.isArray(data) ? data : []
  return list
    .map((item) => (typeof item === 'string' ? item : item?.keyword || item?.word || ''))
    .map((s) => String(s).trim())
    .filter(Boolean)
}

async function loadHotKeywords() {
  if (hotLoading) return
  hotLoading = true
  try {
    const res = await videoApi.getSearchKeywordTop()
    const words = normalizeHotList(res.data)
    hotKeywords.value = words.length ? words : FALLBACK_HOT
  } catch {
    if (!hotKeywords.value.length) hotKeywords.value = FALLBACK_HOT
  } finally {
    hotLoading = false
  }
}

function handleSearch() {
  if (!keyword.value.trim()) return
  showSuggest.value = false
  router.push({ name: 'Search', query: { keyword: keyword.value.trim() } })
}

function searchKeyword(kw) {
  keyword.value = kw
  handleSearch()
}

async function handleLogout() {
  await userStore.logout()
}

function onScroll() {
  isScrolled.value = window.scrollY > 10
}

function onSearchFocus() {
  showSuggest.value = true
  loadHotKeywords()
}

onMounted(async () => {
  window.addEventListener('scroll', onScroll)
  await loadHotKeywords()
})

onUnmounted(() => {
  window.removeEventListener('scroll', onScroll)
})
</script>

<style scoped lang="scss">
.app-header {
  position: sticky;
  top: 0;
  z-index: 100;
  height: var(--bili-header-height);
  background: rgba(15, 17, 23, 0.92);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border-bottom: 1px solid transparent;
  transition: all var(--bili-transition);

  &.scrolled {
    border-bottom-color: var(--bili-border);
    box-shadow: var(--bili-shadow-md);
  }
}

.header-inner {
  display: flex;
  align-items: center;
  height: 100%;
  gap: 24px;
}

.logo {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-shrink: 0;
  transition: transform var(--bili-transition-fast);

  &:hover {
    transform: scale(1.03);
  }

  .logo-icon {
    width: 38px;
    height: 38px;
    filter: drop-shadow(0 2px 4px rgba(99, 102, 241, 0.35));
  }

  .logo-text {
    font-size: 22px;
    font-weight: 700;
    color: var(--bili-pink);
    letter-spacing: -0.5px;
    background: linear-gradient(135deg, #6366f1 0%, #818cf8 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }
}

.search-box {
  flex: 1;
  max-width: 520px;
  position: relative;
}

.search-form {
  display: flex;
  height: 40px;
  border: 1.5px solid var(--bili-border);
  border-radius: var(--bili-radius-lg);
  overflow: hidden;
  background: var(--bili-bg);
  transition: all var(--bili-transition);

  &:focus-within {
    border-color: var(--bili-pink);
    background: var(--bili-white);
    box-shadow: 0 0 0 3px var(--bili-pink-light);
  }
}

.search-input {
  flex: 1;
  border: none;
  background: transparent;
  padding: 0 16px;
  font-size: 14px;
}

.search-btn {
  width: 52px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--bili-text-tertiary);
  transition: all var(--bili-transition-fast);

  &:hover {
    color: var(--bili-pink);
  }
}

.search-suggest {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  right: 0;
  background: var(--bili-white);
  border-radius: var(--bili-radius-lg);
  box-shadow: var(--bili-shadow-lg);
  padding: 12px 0;
  z-index: 200;
  animation: fadeInUp 0.2s ease;
}

.suggest-title {
  display: flex;
  align-items: center;
  padding: 4px 16px 8px;
  font-size: 12px;
  font-weight: 500;
  color: var(--bili-text-secondary);
}

.suggest-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 16px;
  cursor: pointer;
  transition: background var(--bili-transition-fast);

  &:hover {
    background: var(--bili-pink-light);
  }

  .rank {
    width: 20px;
    height: 20px;
    text-align: center;
    font-size: 12px;
    font-weight: 500;
    color: var(--bili-text-tertiary);
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 4px;

    &.top {
      color: var(--bili-white);
      background: var(--bili-pink);
      border-radius: 4px;
    }
  }
}

.search-mask {
  position: fixed;
  inset: 0;
  z-index: 99;
}

.header-nav {
  display: flex;
  align-items: center;
  gap: 4px;
  flex-shrink: 0;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 8px 14px;
  border-radius: var(--bili-radius);
  color: var(--bili-text-secondary);
  font-size: 14px;
  font-weight: 500;
  position: relative;
  transition: all var(--bili-transition-fast);

  &:hover {
    color: var(--bili-pink);
    background: var(--bili-pink-light);
  }
}

.upload-btn {
  background: linear-gradient(135deg, #6366f1 0%, #818cf8 100%);
  color: var(--bili-white) !important;
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.35);

  &:hover {
    background: linear-gradient(135deg, #818cf8 0%, #a5b4fc 100%);
    box-shadow: 0 4px 12px rgba(99, 102, 241, 0.45);
    transform: translateY(-1px);
  }
}

.login-btn {
  color: var(--bili-pink);
  font-weight: 600;
}

.badge {
  position: absolute;
  top: 2px;
  right: 4px;
  min-width: 16px;
  height: 16px;
  padding: 0 4px;
  border-radius: 8px;
  background: var(--bili-danger);
  color: var(--bili-white);
  font-size: 10px;
  line-height: 16px;
  text-align: center;
  font-weight: 600;
}

.user-avatar-wrap {
  position: relative;
  margin-left: 8px;

  &:hover .user-dropdown {
    opacity: 1;
    visibility: visible;
    transform: translateY(0);
  }
}

.user-avatar {
  display: block;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  overflow: hidden;
  border: 2px solid transparent;
  transition: all var(--bili-transition-fast);

  &:hover {
    border-color: var(--bili-pink);
    box-shadow: 0 0 0 3px var(--bili-pink-light);
  }

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}

.user-dropdown {
  position: absolute;
  top: calc(100% + 12px);
  right: -8px;
  min-width: 180px;
  background: var(--bili-white);
  border-radius: var(--bili-radius-lg);
  box-shadow: var(--bili-shadow-lg);
  padding: 0;
  opacity: 0;
  visibility: hidden;
  transform: translateY(-8px);
  transition: all var(--bili-transition);
  z-index: 200;
  overflow: hidden;
}

.dropdown-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 14px 16px;
  background: linear-gradient(135deg, var(--bili-pink-light) 0%, rgba(251, 114, 153, 0.03) 100%);

  .dropdown-avatar {
    width: 32px;
    height: 32px;
    border-radius: 50%;
    object-fit: cover;
  }

  .dropdown-name {
    font-size: 14px;
    font-weight: 600;
    color: var(--bili-text);
  }
}

.dropdown-divider {
  height: 1px;
  background: var(--bili-border-light);
  margin: 0;
}

.dropdown-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  font-size: 13px;
  color: var(--bili-text-secondary);
  transition: all var(--bili-transition-fast);
  width: 100%;

  &:hover {
    background: var(--bili-pink-light);
    color: var(--bili-pink);
  }

  &.logout {
    color: var(--bili-danger);
    &:hover {
      background: rgba(245, 63, 63, 0.06);
      color: var(--bili-danger);
    }
  }

  svg {
    flex-shrink: 0;
  }
}
</style>
