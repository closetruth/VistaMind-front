<template>
  <div class="account-layout container">
    <aside class="account-sidebar">
      <div class="user-card">
        <img :src="avatarUrl" class="avatar" alt="" />
        <p class="nickname">{{ userStore.userInfo?.nickName }}</p>
        <router-link :to="`/user/${userStore.userInfo?.userId}`" class="space-link">
          查看我的主页 →
        </router-link>
      </div>
      <nav class="side-nav">
        <router-link
          v-for="item in menuItems"
          :key="item.path"
          :to="item.path"
          class="side-item"
          :class="{ active: route.path === item.path }"
        >
          <span class="icon">{{ item.icon }}</span>
          <span>{{ item.title }}</span>
          <span v-if="item.badge" class="badge">{{ item.badge }}</span>
        </router-link>
      </nav>
    </aside>
    <main class="account-main">
      <h1 v-if="route.meta.title" class="page-title">{{ route.meta.title }}</h1>
      <router-view />
    </main>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useUserStore } from '@/stores'
import { getAvatarUrl } from '@/utils/format'

const route = useRoute()
const userStore = useUserStore()

const avatarUrl = computed(() => getAvatarUrl(userStore.userInfo?.avatar))

const menuItems = computed(() => [
  { path: '/account/home', title: '概览', icon: '🏠' },
  { path: '/account/statistics', title: '数据中心', icon: '📊' },
  { path: '/account/videos', title: '我的投稿', icon: '🎬' },
  { path: '/account/interact', title: '互动管理', icon: '🛡' },
  { path: '/account/collection', title: '我的收藏', icon: '⭐' },
  { path: '/account/follow', title: '关注粉丝', icon: '👥' },
  { path: '/account/history', title: '历史记录', icon: '🕐' },
  { path: '/account/message', title: '我的消息', icon: '💬' },
  { path: '/account/settings', title: '账号设置', icon: '⚙️' }
])
</script>

<style scoped lang="scss">
.account-layout {
  display: grid;
  grid-template-columns: 240px 1fr;
  gap: 24px;
  padding: 24px 24px 60px;
  align-items: start;
}

.account-sidebar {
  position: sticky;
  top: calc(var(--bili-header-height) + 16px);
  background: var(--bili-white);
  border-radius: var(--bili-radius-lg);
  overflow: hidden;
  box-shadow: var(--bili-shadow-md);
}

.user-card {
  padding: 28px 20px;
  text-align: center;
  background: linear-gradient(180deg, rgba(99, 102, 241, 0.12) 0%, var(--bili-white) 100%);
  border-bottom: 1px solid var(--bili-border);

  .avatar {
    width: 72px;
    height: 72px;
    border-radius: 50%;
    object-fit: cover;
    margin: 0 auto 12px;
    border: 3px solid var(--bili-white);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
  }

  .nickname {
    font-size: 17px;
    font-weight: 700;
    margin-bottom: 8px;
    color: var(--bili-text);
  }

  .space-link {
    font-size: 12px;
    font-weight: 500;
    color: var(--bili-pink);
    transition: opacity var(--bili-transition-fast);

    &:hover {
      opacity: 0.8;
    }
  }
}

.side-nav {
  padding: 10px;
}

.side-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 16px;
  border-radius: var(--bili-radius);
  font-size: 14px;
  font-weight: 500;
  color: var(--bili-text-secondary);
  margin-bottom: 2px;
  transition: all var(--bili-transition-fast);

  &:hover {
    background: var(--bili-pink-light);
    color: var(--bili-pink);
  }

  &.active {
    background: var(--bili-pink-light);
    color: var(--bili-pink);
    font-weight: 600;
  }

  .icon {
    width: 20px;
    text-align: center;
    font-size: 15px;
  }

  .badge {
    margin-left: auto;
    min-width: 18px;
    height: 18px;
    padding: 0 5px;
    border-radius: 9px;
    background: var(--bili-pink);
    color: var(--bili-white);
    font-size: 11px;
    font-weight: 600;
    line-height: 18px;
    text-align: center;
  }
}

.account-main {
  min-height: 400px;
}

.page-title {
  font-size: 22px;
  font-weight: 700;
  margin-bottom: 20px;
  color: var(--bili-text);
}

@media (max-width: 768px) {
  .account-layout {
    grid-template-columns: 1fr;
  }

  .account-sidebar {
    position: static;
  }
}
</style>
