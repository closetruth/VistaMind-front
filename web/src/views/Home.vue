<template>
  <div class="home-page">
    <section v-if="recommendList.length" class="banner-section container">
      <div class="banner-main">
        <router-link
          v-if="recommendList[0]"
          :to="`/video/${recommendList[0].videoId}`"
          class="banner-item main"
        >
          <img :src="getResourceUrl(recommendList[0].videoCover)" :alt="recommendList[0].videoName" />
          <div class="banner-info">
            <h2>{{ recommendList[0].videoName }}</h2>
          </div>
        </router-link>
      </div>
      <div class="banner-side">
        <router-link
          v-for="item in recommendList.slice(1, 5)"
          :key="item.videoId"
          :to="`/video/${item.videoId}`"
          class="banner-item side"
        >
          <img :src="getResourceUrl(item.videoCover)" :alt="item.videoName" />
          <div class="banner-info">
            <p>{{ item.videoName }}</p>
          </div>
        </router-link>
      </div>
    </section>

    <section v-if="hotList.length" class="video-section container">
      <div class="section-header">
        <h2 class="section-title">
          <span class="title-icon">🔥</span>
          热门视频
        </h2>
      </div>
      <div class="video-grid">
        <VideoCard v-for="video in hotList" :key="video.videoId" :video="video" />
      </div>
    </section>

    <section class="video-section container">
      <div class="section-header">
        <h2 class="section-title">
          <span class="title-icon">✨</span>
          推荐视频
        </h2>
        <button v-if="loading" class="refresh-btn" disabled>加载中...</button>
        <button v-else class="refresh-btn" @click="loadVideos(true)">
          <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor">
            <path d="M17.65 6.35A7.958 7.958 0 0 0 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08A5.99 5.99 0 0 1 12 18c-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/>
          </svg>
          换一换
        </button>
      </div>

      <div v-if="loading && !videoList.length" class="loading-spinner">加载中</div>
      <div v-else-if="videoList.length" class="video-grid">
        <VideoCard v-for="video in videoList" :key="video.videoId" :video="video" />
      </div>
      <div v-else class="empty-state">
        <svg viewBox="0 0 24 24" width="48" height="48" fill="var(--bili-text-tertiary)" style="margin-bottom:12px">
          <path d="M15.5 14h-.79l-.28-.27A6.471 6.471 0 0 0 16 9.5 6.5 6.5 0 1 0 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/>
        </svg>
        <p>暂无视频</p>
      </div>

      <div v-if="hasMore && !loading" class="load-more">
        <button class="btn-outline" @click="loadMore">加载更多</button>
      </div>
      <div v-if="loading && videoList.length" class="loading-more">加载中...</div>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import VideoCard from '@/components/video/VideoCard.vue'
import { videoApi } from '@/api'
import { getResourceUrl, normalizeVideoList, buildLoadVideoParams } from '@/utils/format'

const recommendList = ref([])
const hotList = ref([])
const videoList = ref([])
const loading = ref(false)
const pageNo = ref(1)
const hasMore = ref(true)

async function loadRecommend() {
  try {
    const res = await videoApi.loadRecommendVideo()
    recommendList.value = normalizeVideoList(res.data)
  } catch {
    recommendList.value = []
  }
}

async function loadHot() {
  try {
    const res = await videoApi.loadHotVideoList(1)
    hotList.value = normalizeVideoList(res.data)
  } catch {
    hotList.value = []
  }
}

async function loadVideos(reset = false) {
  if (loading.value) return
  loading.value = true
  if (reset) {
    pageNo.value = 1
    hasMore.value = true
  }

  try {
    const res = await videoApi.loadVideo(buildLoadVideoParams({ pageNo: pageNo.value }))
    const payload = res.data || {}
    const list = normalizeVideoList(payload)
    const pageSize = payload.pageSize || 15
    const totalCount = payload.totalCount ?? 0
    const currentPage = payload.pageNo || pageNo.value

    if (reset) {
      videoList.value = list
    } else {
      videoList.value.push(...list)
    }

    hasMore.value = currentPage * pageSize < totalCount
    if (!recommendList.value.length && videoList.value.length) {
      recommendList.value = videoList.value.slice(0, 5)
    }
  } catch {
    if (reset) videoList.value = []
    hasMore.value = false
  } finally {
    loading.value = false
  }
}

function loadMore() {
  pageNo.value++
  loadVideos()
}

onMounted(() => {
  loadRecommend()
  loadHot()
  loadVideos(true)
})
</script>

<style scoped lang="scss">
.banner-section {
  display: flex;
  gap: 12px;
  margin-bottom: 28px;
  height: 300px;
}

.banner-main {
  flex: 1.6;
}

.banner-side {
  flex: 1;
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: 1fr 1fr;
  gap: 12px;
}

.banner-item {
  position: relative;
  display: block;
  border-radius: var(--bili-radius-lg);
  overflow: hidden;
  background: var(--bili-border-light);
  box-shadow: var(--bili-shadow);

  &.main {
    height: 100%;
  }

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  }

  &:hover img {
    transform: scale(1.05);
  }

  &:hover .banner-info {
    opacity: 1;
  }
}

.banner-info {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  padding: 48px 16px 14px;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.75));
  color: #fff;
  opacity: 0.9;
  transition: opacity 0.3s;

  h2 {
    font-size: 17px;
    font-weight: 600;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }

  p {
    font-size: 13px;
    font-weight: 500;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
}

.section-title {
  font-size: 22px;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--bili-text);
}

.title-icon {
  font-size: 20px;
}

.refresh-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  border-radius: var(--bili-radius);
  font-size: 13px;
  font-weight: 500;
  color: var(--bili-text-secondary);
  transition: all var(--bili-transition);

  &:hover:not(:disabled) {
    color: var(--bili-pink);
    background: var(--bili-pink-light);
  }
}

.video-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 20px 16px;
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 20px;
  color: var(--bili-text-tertiary);
  font-size: 15px;
}

.load-more {
  display: flex;
  justify-content: center;
  margin-top: 36px;
}

.loading-more {
  text-align: center;
  padding: 20px;
  color: var(--bili-text-tertiary);
}

@media (max-width: 900px) {
  .banner-section {
    flex-direction: column;
    height: auto;
  }

  .banner-main .banner-item {
    height: 220px;
  }
}
</style>
