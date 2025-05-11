<template>
  <div class="search-page">
    <div class="search-header">
      <div class="back-button" @click="$emit('back')">
        <svg viewBox="0 0 24 24" class="back-icon">
          <path d="M20 11H7.83l5.59-5.59L12 4l-8 8 8 8 1.41-1.41L7.83 13H20v-2z"></path>
        </svg>
      </div>
      <div class="search-input-container">
        <input 
          type="text" 
          class="search-page-input" 
          v-model="searchQuery" 
          placeholder="검색" 
          @input="onSearch"
          ref="searchInput"
          autofocus
        >
        <button 
          v-if="searchQuery.trim()" 
          class="search-clear-button" 
          @click="clearSearch"
        >
          <svg viewBox="0 0 24 24" class="clear-icon">
            <path d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12 19 6.41z"></path>
          </svg>
        </button>
      </div>
    </div>
    
    <div class="search-results">
      <div v-if="loading" class="loading-indicator">
        <div class="spinner"></div>
        <div class="loading-text">검색 중...</div>
      </div>
      
      <div v-else-if="searchQuery.trim() && searchResults.length === 0" class="no-results">
        '{{ searchQuery }}'에 대한 결과가 없습니다
      </div>
      
      <div v-else-if="searchQuery.trim()" class="search-results-list">
        <div 
          v-for="(video, index) in searchResults" 
          :key="index" 
          class="search-result-item"
          @click="selectVideo(video)"
        >
          <div class="search-result-thumbnail">
            <video :src="video.url" v-if="video.url && !video.isRefreshed"></video>
            <div class="thumbnail-placeholder" v-else>
              <svg viewBox="0 0 24 24" class="video-placeholder-icon">
                <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 14.5v-9l6 4.5-6 4.5z"></path>
              </svg>
            </div>
          </div>
          <div class="search-result-info">
            <div class="search-result-title">{{ video.title }}</div>
            <div class="search-result-meta">{{ formatDate(video.uploadDate) }} • {{ formatFileSize(video.size) }}</div>
          </div>
        </div>
      </div>
      
      <div v-else class="search-suggestions">
        <div class="search-history" v-if="searchHistory.length > 0">
          <h3 class="section-title">최근 검색어</h3>
          <div 
            v-for="(query, index) in searchHistory" 
            :key="index" 
            class="history-item"
            @click="applySearch(query)"
          >
            <svg viewBox="0 0 24 24" class="history-icon">
              <path d="M13 3c-4.97 0-9 4.03-9 9H1l3.89 3.89.07.14L9 12H6c0-3.87 3.13-7 7-7s7 3.13 7 7-3.13 7-7 7c-1.93 0-3.68-.79-4.94-2.06l-1.42 1.42C8.27 19.99 10.51 21 13 21c4.97 0 9-4.03 9-9s-4.03-9-9-9zm-1 5v5l4.28 2.54.72-1.21-3.5-2.08V8H12z"></path>
            </svg>
            <span class="history-text">{{ query }}</span>
          </div>
        </div>
        
        <div class="trending-searches">
          <h3 class="section-title">인기 검색어</h3>
          <div 
            v-for="(trend, index) in trendingSearches" 
            :key="index" 
            class="trend-item"
            @click="applySearch(trend)"
          >
            <svg viewBox="0 0 24 24" class="trend-icon">
              <path d="M17.38 10.79l-2.2-2.2c-.28-.28-.36-.67-.25-1.02.37-1.12.57-2.32.57-3.57 0-.55.45-1 1-1H20c.55 0 1 .45 1 1 0 9.39-7.61 17-17 17-.55 0-1-.45-1-1v-3.49c0-.55.45-1 1-1 1.24 0 2.45-.2 3.57-.57.1-.03.2-.05.31-.05.26 0 .51.1.71.29l2.2 2.2c2.83-1.45 5.15-3.76 6.59-6.59z"></path>
            </svg>
            <span class="trend-text">{{ trend }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, computed, watch } from 'vue'

export default defineComponent({
  name: 'SearchPage',
  props: {
    videos: {
      type: Array,
      required: true
    }
  },
  emits: ['back', 'selectVideo'],
  setup(props, { emit }) {
    const searchQuery = ref('');
    const loading = ref(false);
    const searchInput = ref<HTMLInputElement | null>(null);
    const searchHistory = ref<string[]>([]);
    const trendingSearches = ref<string[]>([
      '인기 동영상',
      '새로운 동영상',
      '추천 영상',
      'Top 10',
      '최신 업로드'
    ]);
    
    // 검색 결과
    const searchResults = computed(() => {
      if (!searchQuery.value.trim()) return [];
      
      const query = searchQuery.value.trim().toLowerCase();
      return props.videos.filter(video => 
        video.title.toLowerCase().includes(query)
      );
    });
    
    // 검색 실행
    const onSearch = () => {
      if (loading.value) return;
      
      if (searchQuery.value.trim()) {
        loading.value = true;
        
        // 검색 시뮬레이션(실제 API 호출 대신)
        setTimeout(() => {
          loading.value = false;
        }, 300);
      }
    };
    
    // 검색어 초기화
    const clearSearch = () => {
      searchQuery.value = '';
      if (searchInput.value) {
        searchInput.value.focus();
      }
    };
    
    // 비디오 선택
    const selectVideo = (video) => {
      emit('selectVideo', video);
      saveSearchToHistory(searchQuery.value);
    };
    
    // 검색어 적용
    const applySearch = (query: string) => {
      searchQuery.value = query;
      onSearch();
      saveSearchToHistory(query);
    };
    
    // 검색 기록 저장
    const saveSearchToHistory = (query: string) => {
      if (!query.trim()) return;
      
      // 이미 있는 검색어는 제거
      const index = searchHistory.value.indexOf(query);
      if (index > -1) {
        searchHistory.value.splice(index, 1);
      }
      
      // 최근 검색어 맨 앞에 추가
      searchHistory.value.unshift(query);
      
      // 최대 5개만 유지
      if (searchHistory.value.length > 5) {
        searchHistory.value.pop();
      }
      
      // 로컬 스토리지에 저장
      localStorage.setItem('searchHistory', JSON.stringify(searchHistory.value));
    };
    
    // 포맷 함수
    const formatDate = (date: Date): string => {
      const now = new Date();
      const diffTime = Math.abs(now.getTime() - new Date(date).getTime());
      const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24));
      
      if (diffDays === 0) {
        return '오늘';
      } else if (diffDays === 1) {
        return '어제';
      } else if (diffDays < 7) {
        return `${diffDays}일 전`;
      } else if (diffDays < 30) {
        return `${Math.floor(diffDays / 7)}주 전`;
      } else {
        return `${Math.floor(diffDays / 30)}개월 전`;
      }
    };
    
    const formatFileSize = (bytes: number): string => {
      if (bytes === 0) return '0 Bytes';
      
      const sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB'];
      const i = Math.floor(Math.log(bytes) / Math.log(1024));
      
      return parseFloat((bytes / Math.pow(1024, i)).toFixed(2)) + ' ' + sizes[i];
    };
    
    // 컴포넌트 마운트 시 검색 기록 로드
    onMounted(() => {
      const savedHistory = localStorage.getItem('searchHistory');
      if (savedHistory) {
        try {
          searchHistory.value = JSON.parse(savedHistory);
        } catch (e) {
          console.error('검색 기록을 불러오는 중 오류가 발생했습니다:', e);
        }
      }
      
      // 검색 입력창에 포커스
      if (searchInput.value) {
        searchInput.value.focus();
      }
    });
    
    return {
      searchQuery,
      searchResults,
      loading,
      searchInput,
      searchHistory,
      trendingSearches,
      onSearch,
      clearSearch,
      selectVideo,
      applySearch,
      formatDate,
      formatFileSize
    };
  }
});
</script>

<style scoped>
.search-page {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: white;
  z-index: 1000;
  display: flex;
  flex-direction: column;
}

.search-header {
  display: flex;
  align-items: center;
  padding: 8px 16px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  background-color: white;
}

.back-button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  cursor: pointer;
  margin-right: 8px;
}

.back-button:hover {
  background-color: rgba(0, 0, 0, 0.05);
  border-radius: 50%;
}

.back-icon {
  width: 24px;
  height: 24px;
  fill: #606060;
}

.search-input-container {
  flex: 1;
  position: relative;
}

.search-page-input {
  width: 100%;
  height: 40px;
  padding: 0 40px 0 16px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 20px;
  outline: none;
}

.search-page-input:focus {
  border-color: #1c62b9;
  box-shadow: 0 0 0 1px #1c62b9;
}

.search-clear-button {
  position: absolute;
  right: 8px;
  top: 50%;
  transform: translateY(-50%);
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
}

.clear-icon {
  width: 18px;
  height: 18px;
  fill: #606060;
}

.search-results {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
}

.loading-indicator {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 32px;
}

.spinner {
  width: 32px;
  height: 32px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #1c62b9;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 16px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-text {
  color: #606060;
}

.no-results {
  padding: 32px 16px;
  text-align: center;
  color: #606060;
  font-size: 14px;
}

.search-results-list {
  display: flex;
  flex-direction: column;
}

.search-result-item {
  display: flex;
  padding: 12px 0;
  cursor: pointer;
  border-bottom: 1px solid #f0f0f0;
}

.search-result-item:hover {
  background-color: #f8f8f8;
}

.search-result-thumbnail {
  width: 120px;
  height: 68px;
  margin-right: 12px;
  background-color: #f0f0f0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  overflow: hidden;
}

.search-result-thumbnail video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.video-placeholder-icon {
  width: 32px;
  height: 32px;
  fill: #606060;
}

.search-result-info {
  flex: 1;
}

.search-result-title {
  font-size: 14px;
  font-weight: 500;
  color: #030303;
  margin-bottom: 4px;
}

.search-result-meta {
  font-size: 12px;
  color: #606060;
}

.section-title {
  font-size: 16px;
  font-weight: 500;
  color: #030303;
  margin: 24px 0 16px;
}

.search-history {
  margin-bottom: 32px;
}

.history-item,
.trend-item {
  display: flex;
  align-items: center;
  padding: 12px 0;
  cursor: pointer;
}

.history-item:hover,
.trend-item:hover {
  background-color: #f8f8f8;
}

.history-icon,
.trend-icon {
  width: 18px;
  height: 18px;
  fill: #606060;
  margin-right: 12px;
}

.history-text,
.trend-text {
  font-size: 14px;
  color: #030303;
}
</style> 