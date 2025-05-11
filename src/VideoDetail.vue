<template>
  <div class="video-detail-overlay" @click="closeDetail"></div>
  <div class="video-detail-modal" :class="{ 'shorts-modal': isShort }">
    <div class="modal-header">
      <h2>{{ video.title }}</h2>
      <button class="close-button" @click="$emit('close')">×</button>
    </div>
    
    <div class="modal-content">
      <div class="video-player" :class="{ 'shorts-player': isShort }">
        <video :src="video.url" controls v-if="video.url && !video.isRefreshed"></video>
        <div class="thumbnail-placeholder" v-else>
          <div class="refresh-notice" v-if="video.isRefreshed">
            <svg viewBox="0 0 24 24" class="refresh-icon">
              <path d="M17.65 6.35C16.2 4.9 14.21 4 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08c-.82 2.33-3.04 4-5.65 4-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 9h7V2l-2.35 4.35z"/>
            </svg>
            <div class="refresh-text">
              <div>새로고침 후 재생할 수 없습니다.</div>
              <div>새 비디오를 업로드하세요.</div>
            </div>
          </div>
          <svg v-else viewBox="0 0 24 24" class="video-placeholder-icon">
            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 14.5v-9l6 4.5-6 4.5z"></path>
          </svg>
        </div>
      </div>
      
      <!-- 쇼츠인 경우 태그 표시 -->
      <div class="video-type-tag" v-if="isShort">쇼츠</div>
      
      <div class="video-info">
        <h3 class="video-title">{{ video.title }}</h3>
        <div class="video-meta">
          <span class="upload-date">{{ formatDate(video.uploadDate) }}</span>
          <span class="video-size">{{ formatFileSize(video.size) }}</span>
          <div class="video-actions">
            <button class="subscribe-button" @click="subscribeToChannel" :class="{ 'subscribed': isSubscribed }">
              {{ isSubscribed ? '구독중' : '구독' }}
            </button>
            <button class="watch-later-button" @click="addToWatchLater">
              <svg viewBox="0 0 24 24" class="watch-later-icon">
                <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8zm.5-13H11v6l5.25 3.15.75-1.23-4.5-2.67z"/>
              </svg>
              나중에 보기
            </button>
          </div>
        </div>
      </div>
      
      <div class="comments-section">
        <h3 class="comments-title">댓글 {{ comments.length }}개</h3>
        
        <div class="comment-form">
          <textarea 
            v-model="newComment" 
            placeholder="댓글 추가..." 
            class="comment-input"
            :disabled="isSubmitting"
            @focus="isCommentFocused = true"
          ></textarea>
          
          <div class="comment-actions" v-if="isCommentFocused">
            <button class="cancel-button" @click="cancelComment">취소</button>
            <button 
              class="submit-button" 
              :disabled="!newComment.trim() || isSubmitting"
              @click="addComment"
            >
              댓글
            </button>
          </div>
        </div>
        
        <div class="comments-list">
          <div v-if="comments.length === 0" class="no-comments">
            첫 번째 댓글을 남겨보세요!
          </div>
          
          <div v-for="(comment, index) in comments" :key="index" class="comment-item">
            <div class="comment-avatar">
              <div class="avatar-circle">
                {{ getInitials(comment.author) }}
              </div>
            </div>
            <div class="comment-content">
              <div class="comment-header">
                <span class="comment-author">{{ comment.author }}</span>
                <span class="comment-date">{{ formatCommentDate(comment.date) }}</span>
                <button class="delete-button" @click="deleteComment(index)" title="댓글 삭제">
                  <svg viewBox="0 0 24 24" class="delete-icon">
                    <path d="M6 19c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7H6v12zM19 4h-3.5l-1-1h-5l-1 1H5v2h14V4z"></path>
                  </svg>
                </button>
              </div>
              <div class="comment-text">{{ comment.text }}</div>
              <div class="comment-actions-row">
                <button class="like-button" @click="toggleLike(index)">
                  <svg viewBox="0 0 24 24" class="thumb-icon" :class="{ 'active': comment.liked }">
                    <path d="M1 21h4V9H1v12zm22-11c0-1.1-.9-2-2-2h-6.31l.95-4.57.03-.32c0-.41-.17-.79-.44-1.06L14.17 1 7.59 7.59C7.22 7.95 7 8.45 7 9v10c0 1.1.9 2 2 2h9c.83 0 1.54-.5 1.84-1.22l3.02-7.05c.09-.23.14-.47.14-.73v-1.91l-.01-.01L23 10z"></path>
                  </svg>
                  <span>{{ comment.likes }}</span>
                </button>
                <button class="dislike-button" @click="toggleDislike(index)">
                  <svg viewBox="0 0 24 24" class="thumb-icon" :class="{ 'active': comment.disliked }">
                    <path d="M15 3H6c-.83 0-1.54.5-1.84 1.22l-3.02 7.05c-.09.23-.14.47-.14.73v1.91l.01.01L1 14c0 1.1.9 2 2 2h6.31l-.95 4.57-.03.32c0 .41.17.79.44 1.06L9.83 23l6.59-6.59c.36-.36.58-.86.58-1.41V5c0-1.1-.9-2-2-2zm4 0v12h4V3h-4z"></path>
                  </svg>
                  <span>{{ comment.dislikes }}</span>
                </button>
                <button class="reply-button">답글</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  
  <!-- 댓글 삭제 확인 모달 -->
  <div class="delete-modal-overlay" v-if="showDeleteModal" @click="cancelDelete"></div>
  <div class="delete-modal" v-if="showDeleteModal">
    <div class="delete-modal-content">
      <h3>댓글 삭제</h3>
      <p>이 댓글을 삭제하시겠습니까?</p>
      <div class="delete-modal-buttons">
        <button class="cancel-delete-button" @click="cancelDelete">취소</button>
        <button class="confirm-delete-button" @click="confirmDelete">삭제</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, onMounted } from 'vue'

export default defineComponent({
  name: 'VideoDetail',
  props: {
    video: {
      type: Object,
      required: true
    }
  },
  emits: ['close'],
  setup(props, { emit }) {
    const newComment = ref('');
    const isCommentFocused = ref(false);
    const isSubmitting = ref(false);
    const comments = ref([]);
    const showDeleteModal = ref(false);
    const commentToDelete = ref(-1);
    const isSubscribed = ref(false);
    
    // 로컬 스토리지에서 댓글과 구독 정보 불러오기
    onMounted(() => {
      loadComments();
      checkSubscriptionStatus();
      
      // 동영상이 재생되기 전에도 시청 기록에 표시
      const videoPlayer = document.querySelector('.video-player video');
      if (videoPlayer) {
        videoPlayer.addEventListener('play', () => {
          // 이미 App.vue에서 시청 기록에 추가했지만, 
          // 동영상이 실제로 재생되는 시점의 시간을 기록하기 위한 이벤트
          const watchHistoryEvent = new CustomEvent('videoPlayed', {
            detail: { video: props.video }
          });
          document.dispatchEvent(watchHistoryEvent);
        });
      }
    });
    
    const loadComments = () => {
      const savedComments = localStorage.getItem(`comments_${props.video.id}`);
      if (savedComments) {
        try {
          const parsedComments = JSON.parse(savedComments);
          // 날짜 객체 복원 및 dislikes 속성이 없는 경우 추가
          comments.value = parsedComments.map(comment => ({
            ...comment,
            date: new Date(comment.date),
            dislikes: comment.dislikes || 0 // dislikes 속성이 없는 경우 기본값 0 설정
          }));
        } catch (e) {
          console.error('댓글 데이터를 불러오는 중 오류가 발생했습니다:', e);
        }
      }
    };
    
    const saveComments = () => {
      try {
        localStorage.setItem(`comments_${props.video.id}`, JSON.stringify(comments.value));
      } catch (e) {
        console.error('댓글 데이터를 저장하는 중 오류가 발생했습니다:', e);
      }
    };
    
    const closeDetail = (event) => {
      // 오버레이 클릭 시에만 닫기 (모달 내부 클릭 시에는 닫지 않음)
      if (event.target.classList.contains('video-detail-overlay')) {
        event.stopPropagation();
        event.preventDefault();
        emit('close');
      }
    };
    
    const cancelComment = () => {
      newComment.value = '';
      isCommentFocused.value = false;
    };
    
    const addComment = () => {
      if (!newComment.value.trim()) return;
      
      isSubmitting.value = true;
      
      // 댓글 추가 (실제로는 API 호출이 필요할 수 있음)
      setTimeout(() => {
        const comment = {
          author: '사용자', // 실제로는 로그인한 사용자 정보
          text: newComment.value,
          date: new Date(),
          likes: 0,
          dislikes: 0,
          liked: false,
          disliked: false
        };
        
        comments.value.unshift(comment);
        saveComments();
        
        newComment.value = '';
        isSubmitting.value = false;
        isCommentFocused.value = false;
      }, 300);
    };
    
    const toggleLike = (index) => {
      const comment = comments.value[index];
      
      if (comment.liked) {
        // 이미 좋아요를 눌렀다면 취소
        comment.likes--;
        comment.liked = false;
      } else {
        // 좋아요 추가
        comment.likes++;
        comment.liked = true;
        
        // 싫어요를 눌렀었다면 싫어요 취소
        if (comment.disliked) {
          comment.dislikes--;
          comment.disliked = false;
        }
      }
      
      saveComments();
    };
    
    const toggleDislike = (index) => {
      const comment = comments.value[index];
      
      if (comment.disliked) {
        // 이미 싫어요를 눌렀다면 취소
        comment.dislikes--;
        comment.disliked = false;
      } else {
        // 싫어요 추가
        comment.dislikes++;
        comment.disliked = true;
        
        // 좋아요를 눌렀었다면 좋아요 취소
        if (comment.liked) {
          comment.likes--;
          comment.liked = false;
        }
      }
      
      saveComments();
    };
    
    const deleteComment = (index) => {
      commentToDelete.value = index;
      showDeleteModal.value = true;
    };
    
    const cancelDelete = () => {
      showDeleteModal.value = false;
      commentToDelete.value = -1;
    };
    
    const confirmDelete = () => {
      if (commentToDelete.value > -1) {
        comments.value.splice(commentToDelete.value, 1);
        saveComments();
        showDeleteModal.value = false;
        commentToDelete.value = -1;
      }
    };
    
    const getInitials = (name) => {
      return name.charAt(0).toUpperCase();
    };
    
    const formatDate = (date) => {
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
    
    const formatCommentDate = (date) => {
      const now = new Date();
      const diffTime = Math.abs(now.getTime() - new Date(date).getTime());
      const diffMinutes = Math.floor(diffTime / (1000 * 60));
      
      if (diffMinutes < 1) {
        return '방금 전';
      } else if (diffMinutes < 60) {
        return `${diffMinutes}분 전`;
      } else if (diffMinutes < 1440) {
        return `${Math.floor(diffMinutes / 60)}시간 전`;
      } else {
        return `${Math.floor(diffMinutes / 1440)}일 전`;
      }
    };
    
    const formatFileSize = (bytes) => {
      if (bytes === 0) return '0 Bytes';
      
      const sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB'];
      const i = Math.floor(Math.log(bytes) / Math.log(1024));
      
      return parseFloat((bytes / Math.pow(1024, i)).toFixed(2)) + ' ' + sizes[i];
    };
    
    const addToWatchLater = () => {
      // 사용자 정의 이벤트를 발생시켜 App.vue에서 처리
      const event = new CustomEvent('addToWatchLater', {
        detail: { video: props.video }
      });
      document.dispatchEvent(event);
    };
    
    // 비디오 ID에서 채널 정보 가져오기
    const getChannelInfo = () => {
      // 간단한 구현: 비디오 ID에서 채널 ID를 생성
      const channelId = 'channel_' + props.video.id.split('_')[1];
      return {
        id: channelId,
        name: '채널 ' + props.video.id.split('_')[1]
      };
    };
    
    // 현재 채널 구독 상태 확인
    const checkSubscriptionStatus = () => {
      const savedSubscriptions = localStorage.getItem('subscribedChannels');
      if (savedSubscriptions) {
        try {
          const parsedSubscriptions = JSON.parse(savedSubscriptions);
          const channelInfo = getChannelInfo();
          isSubscribed.value = parsedSubscriptions.some(
            channel => channel.id === channelInfo.id
          );
        } catch (e) {
          console.error('구독 정보를 확인하는 중 오류가 발생했습니다:', e);
        }
      }
    };
    
    // 채널 구독 토글
    const subscribeToChannel = () => {
      const channelInfo = getChannelInfo();
      
      // 사용자 정의 이벤트를 발생시켜 App.vue에서 처리
      const event = new CustomEvent('subscribeChannel', {
        detail: { channel: channelInfo }
      });
      document.dispatchEvent(event);
      
      // 구독 상태 업데이트
      isSubscribed.value = true;
    };
    
    // 쇼츠 여부 확인
    const isShort = computed(() => {
      return props.video.isShort === true || (props.video.duration && props.video.duration <= 50);
    });
    
    return {
      newComment,
      isCommentFocused,
      isSubmitting,
      comments,
      showDeleteModal,
      commentToDelete,
      isSubscribed,
      closeDetail,
      cancelComment,
      addComment,
      toggleLike,
      toggleDislike,
      deleteComment,
      cancelDelete,
      confirmDelete,
      getInitials,
      formatDate,
      formatCommentDate,
      formatFileSize,
      addToWatchLater,
      subscribeToChannel,
      isShort,
    };
  }
});
</script>

<style scoped>
.video-detail-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  z-index: 1000;
}

.video-detail-modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100%;
  max-width: 800px;
  max-height: 90vh;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  z-index: 1001;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 24px;
  border-bottom: 1px solid #e5e5e5;
}

.modal-header h2 {
  font-size: 18px;
  font-weight: 500;
  margin: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.close-button {
  background: transparent;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #606060;
}

.modal-content {
  padding: 0;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.video-player {
  width: 100%;
  position: relative;
  padding-bottom: 56.25%; /* 16:9 비율 */
  height: 0;
  background-color: #000;
}

.video-player video {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.thumbnail-placeholder {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f5f5f5;
}

.refresh-notice {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 16px;
  text-align: center;
}

.refresh-icon {
  width: 40px;
  height: 40px;
  fill: #606060;
  margin-bottom: 12px;
}

.refresh-text {
  color: #606060;
  font-size: 14px;
  line-height: 1.4;
}

.video-placeholder-icon {
  width: 48px;
  height: 48px;
  fill: #606060;
}

.video-info {
  padding: 16px 24px;
  border-bottom: 1px solid #e5e5e5;
}

.video-title {
  font-size: 18px;
  font-weight: 500;
  margin: 0 0 8px 0;
  color: #030303;
}

.video-meta {
  display: flex;
  align-items: center;
  color: #606060;
  font-size: 14px;
  flex-wrap: wrap;
}

.upload-date {
  margin-right: 12px;
}

.video-size {
  margin-right: auto;
}

.video-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.subscribe-button {
  background-color: #cc0000;
  color: white;
  border: none;
  border-radius: 2px;
  padding: 10px 16px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s;
}

.subscribe-button:hover {
  background-color: #aa0000;
}

.subscribe-button.subscribed {
  background-color: #f0f0f0;
  color: #606060;
}

.subscribe-button.subscribed:hover {
  background-color: #e0e0e0;
}

.watch-later-button {
  display: flex;
  align-items: center;
  background-color: #f0f0f0;
  border: none;
  border-radius: 18px;
  padding: 6px 12px;
  cursor: pointer;
  font-size: 14px;
  color: #606060;
  transition: background-color 0.2s;
}

.watch-later-button:hover {
  background-color: #e0e0e0;
}

.watch-later-icon {
  width: 18px;
  height: 18px;
  fill: #606060;
  margin-right: 6px;
}

.comments-section {
  padding: 16px 24px;
}

.comments-title {
  font-size: 16px;
  font-weight: 500;
  margin: 0 0 16px 0;
  color: #030303;
}

.comment-form {
  margin-bottom: 24px;
}

.comment-input {
  width: 100%;
  min-height: 24px;
  max-height: 120px;
  padding: 8px 0;
  font-size: 14px;
  border: none;
  border-bottom: 1px solid #e5e5e5;
  resize: none;
  outline: none;
  font-family: inherit;
}

.comment-input:focus {
  border-bottom-color: #1c62b9;
}

.comment-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 8px;
}

.cancel-button,
.submit-button {
  padding: 8px 16px;
  font-size: 14px;
  border: none;
  border-radius: 2px;
  cursor: pointer;
  margin-left: 8px;
}

.cancel-button {
  background-color: transparent;
  color: #606060;
}

.cancel-button:hover {
  background-color: #f0f0f0;
}

.submit-button {
  background-color: #1c62b9;
  color: white;
}

.submit-button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.comments-list {
  margin-top: 16px;
}

.no-comments {
  text-align: center;
  color: #606060;
  padding: 24px 0;
}

.comment-item {
  display: flex;
  margin-bottom: 24px;
}

.comment-avatar {
  margin-right: 16px;
  flex-shrink: 0;
}

.avatar-circle {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #1c62b9;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 500;
  font-size: 16px;
}

.comment-content {
  flex: 1;
}

.comment-header {
  margin-bottom: 4px;
  display: flex;
  align-items: center;
}

.comment-author {
  font-weight: 500;
  font-size: 14px;
  color: #030303;
  margin-right: 8px;
}

.comment-date {
  font-size: 12px;
  color: #606060;
  flex: 1;
}

.delete-button {
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0.5;
  transition: opacity 0.2s;
}

.delete-button:hover {
  opacity: 1;
}

.delete-icon {
  width: 16px;
  height: 16px;
  fill: #606060;
}

.comment-text {
  font-size: 14px;
  color: #030303;
  margin-bottom: 8px;
  line-height: 1.4;
  white-space: pre-wrap;
}

.comment-actions-row {
  display: flex;
  align-items: center;
}

.like-button,
.dislike-button,
.reply-button {
  background: transparent;
  border: none;
  display: flex;
  align-items: center;
  font-size: 12px;
  color: #606060;
  padding: 4px 8px;
  cursor: pointer;
  margin-right: 8px;
}

.like-button:hover,
.dislike-button:hover {
  color: #030303;
}

.thumb-icon {
  width: 16px;
  height: 16px;
  fill: #606060;
  margin-right: 4px;
}

.thumb-icon.active {
  fill: #1c62b9;
}

.reply-button {
  font-weight: 500;
}

/* 삭제 확인 모달 스타일 */
.delete-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1002;
}

.delete-modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 300px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  z-index: 1003;
}

.delete-modal-content {
  padding: 24px;
}

.delete-modal-content h3 {
  font-size: 18px;
  font-weight: 500;
  margin: 0 0 16px 0;
  color: #030303;
}

.delete-modal-content p {
  font-size: 14px;
  color: #606060;
  margin-bottom: 24px;
}

.delete-modal-buttons {
  display: flex;
  justify-content: flex-end;
}

.cancel-delete-button,
.confirm-delete-button {
  padding: 8px 16px;
  font-size: 14px;
  border: none;
  border-radius: 2px;
  cursor: pointer;
  margin-left: 8px;
}

.cancel-delete-button {
  background-color: transparent;
  color: #606060;
}

.cancel-delete-button:hover {
  background-color: #f0f0f0;
}

.confirm-delete-button {
  background-color: #ff0000;
  color: white;
}

/* 쇼츠 모달 스타일 */
.shorts-modal {
  max-width: 500px;
}

.shorts-player {
  padding-bottom: 177.78%; /* 9:16 비율 (쇼츠 비율) */
  width: 100%;
  max-width: 500px;
  margin: 0 auto;
  border-radius: 12px;
  overflow: hidden;
}

.video-player video {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.shorts-player video {
  object-fit: cover;
}

.video-type-tag {
  display: inline-block;
  background-color: #f00;
  color: white;
  font-size: 14px;
  font-weight: bold;
  padding: 4px 10px;
  border-radius: 4px;
  margin: 16px 24px 0;
}
</style> 