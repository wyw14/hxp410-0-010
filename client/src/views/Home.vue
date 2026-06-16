<template>
  <div class="home-container">
    <div class="card secret-card">
      <div class="card-header">
        <span class="icon">💫</span>
        <h2>今日被宽恕的秘密</h2>
      </div>

      <div v-if="loading" class="loading">
        <div class="spinner"></div>
        <p>正在寻找一段温暖的秘密...</p>
      </div>

      <div v-else-if="!hasSecret" class="empty-state">
        <span class="empty-icon">🌸</span>
        <p>{{ message }}</p>
        <button class="btn btn-primary" @click="goToConfess">
          分享第一个秘密
        </button>
      </div>

      <transition name="fade" v-else>
        <div class="secret-content" :key="secret?.id">
          <p class="secret-text">"{{ secret.content }}"</p>
          <div class="secret-footer">
            <span class="status-badge">{{ secret.status }}</span>
            <button class="btn btn-secondary refresh-btn" @click="fetchRandomSecret">
              🔄 换一个
            </button>
          </div>
          <div class="like-section">
            <button 
              class="like-btn" 
              :class="{ 'liked': isLiked, 'animating': isAnimating }"
              :disabled="isLiked || isSubmitting"
              @click="handleLike"
            >
              <span class="like-icon">{{ isLiked ? '💛' : '🤍' }}</span>
              <span class="like-text">{{ isLiked ? '已理解' : '我理解你' }}</span>
              <span class="like-count">{{ likeCount }} 人理解</span>
            </button>
          </div>
        </div>
      </transition>

      <div class="card-actions">
        <button class="btn btn-primary" @click="goToConfess">
          我也想倾诉
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const loading = ref(true)
const hasSecret = ref(false)
const secret = ref(null)
const message = ref('')
const isSubmitting = ref(false)
const isAnimating = ref(false)
const likeCount = ref(0)
const isLiked = ref(false)

const LIKED_KEY = 'confession_liked_secrets'

function getLikedSecrets() {
  try {
    const liked = localStorage.getItem(LIKED_KEY)
    return liked ? JSON.parse(liked) : []
  } catch {
    return []
  }
}

function setLikedSecret(secretId) {
  try {
    const liked = getLikedSecrets()
    if (!liked.includes(secretId)) {
      liked.push(secretId)
      localStorage.setItem(LIKED_KEY, JSON.stringify(liked))
    }
  } catch (e) {
    console.error('保存点赞记录失败:', e)
  }
}

function checkIsLiked(secretId) {
  const liked = getLikedSecrets()
  return liked.includes(secretId)
}

watch(secret, (newSecret) => {
  if (newSecret) {
    likeCount.value = newSecret.likeCount || 0
    isLiked.value = checkIsLiked(newSecret.id)
  }
})

async function fetchRandomSecret() {
  loading.value = true
  isAnimating.value = false
  try {
    const response = await fetch('/api/secrets/random')
    const data = await response.json()
    hasSecret.value = data.hasSecret
    secret.value = data.secret
    message.value = data.message
  } catch (error) {
    console.error('获取秘密失败:', error)
    hasSecret.value = false
    message.value = '暂时无法连接到服务器'
  } finally {
    loading.value = false
  }
}

async function handleLike() {
  if (isLiked.value || isSubmitting.value || !secret.value) return

  isSubmitting.value = true
  isAnimating.value = true

  try {
    const response = await fetch(`/api/secrets/${secret.value.id}/like`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      }
    })
    const data = await response.json()

    if (data.success) {
      likeCount.value = data.likeCount
      setLikedSecret(secret.value.id)
      isLiked.value = true
    }
  } catch (error) {
    console.error('点亮失败:', error)
  } finally {
    isSubmitting.value = false
    setTimeout(() => {
      isAnimating.value = false
    }, 600)
  }
}

function goToConfess() {
  router.push('/confess')
}

onMounted(() => {
  fetchRandomSecret()
})
</script>

<style scoped>
.home-container {
  width: 100%;
  max-width: 600px;
}

.secret-card {
  animation: slideUp 0.6s ease;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card-header {
  text-align: center;
  margin-bottom: 30px;
}

.icon {
  font-size: 48px;
  display: block;
  margin-bottom: 10px;
}

.card-header h2 {
  color: #333;
  font-size: 24px;
  font-weight: 600;
}

.loading {
  text-align: center;
  padding: 60px 20px;
  color: #666;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid rgba(102, 126, 234, 0.3);
  border-top-color: #667eea;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 20px;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.empty-state {
  text-align: center;
  padding: 40px 20px;
}

.empty-icon {
  font-size: 64px;
  display: block;
  margin-bottom: 20px;
}

.empty-state p {
  color: #666;
  font-size: 16px;
  margin-bottom: 30px;
}

.secret-content {
  padding: 20px 0;
}

.secret-text {
  font-size: 20px;
  line-height: 1.8;
  color: #333;
  font-style: italic;
  text-align: center;
  margin-bottom: 30px;
  padding: 0 10px;
}

.secret-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 15px;
}

.status-badge {
  background: linear-gradient(135deg, #84fab0 0%, #8fd3f4 100%);
  color: #2d5a4a;
  padding: 8px 20px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 500;
}

.refresh-btn {
  padding: 8px 20px;
  font-size: 14px;
  color: #666;
  background: #f0f0f0;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.refresh-btn:hover {
  background: #e0e0e0;
  transform: translateY(-1px);
}

.like-section {
  margin-top: 25px;
  display: flex;
  justify-content: center;
}

.like-btn {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 28px;
  border: 2px solid #e0e0e0;
  background: #fff;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 15px;
  color: #666;
}

.like-btn:hover:not(:disabled) {
  border-color: #ffd93d;
  background: #fffbe6;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(255, 217, 61, 0.3);
}

.like-btn:disabled {
  cursor: default;
}

.like-btn.liked {
  border-color: #ffd93d;
  background: linear-gradient(135deg, #fff9e6 0%, #fff3cd 100%);
  color: #f5a623;
}

.like-icon {
  font-size: 22px;
  transition: transform 0.3s ease;
}

.like-btn.animating .like-icon {
  animation: heartbeat 0.6s ease;
}

@keyframes heartbeat {
  0% { transform: scale(1); }
  25% { transform: scale(1.4); }
  50% { transform: scale(1.1); }
  75% { transform: scale(1.3); }
  100% { transform: scale(1); }
}

.like-text {
  font-weight: 500;
}

.like-count {
  font-size: 13px;
  color: #999;
  padding-left: 10px;
  border-left: 1px solid #e0e0e0;
}

.like-btn.liked .like-count {
  color: #f5a623;
  border-left-color: #ffd93d;
}

.card-actions {
  margin-top: 40px;
  text-align: center;
  padding-top: 30px;
  border-top: 1px solid #eee;
}
</style>
