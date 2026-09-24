<script setup>
import { ref, computed } from 'vue'

const emit = defineEmits(['back'])

const inputToken = ref('eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjE3MzU2ODk2MDB9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c')

const clearAll = () => {
  inputToken.value = ''
}

const decodeBase64Url = (str) => {
  // Convert Base64Url to Base64
  let base64 = str.replace(/-/g, '+').replace(/_/g, '/')
  // Pad with '='
  while (base64.length % 4) {
    base64 += '='
  }
  
  try {
    return JSON.parse(decodeURIComponent(escape(window.atob(base64))))
  } catch (e) {
    return null
  }
}

const decodedData = computed(() => {
  const token = inputToken.value.trim()
  if (!token) return null

  const parts = token.split('.')
  if (parts.length !== 3) return { error: '無效的 JWT 格式 (必須包含三個部分)' }

  const header = decodeBase64Url(parts[0])
  const payload = decodeBase64Url(parts[1])

  if (!header || !payload) return { error: '解析失敗，可能不是有效的 Base64 編碼' }

  return {
    header,
    payload,
    signature: parts[2]
  }
})

const formatTimestamp = (ts) => {
  if (!ts) return ''
  const date = new Date(ts * 1000)
  return date.toLocaleString()
}
</script>

<template>
  <div class="glass-card jwt-card">
    <button class="btn-back" @click="emit('back')">
      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <line x1="19" y1="12" x2="5" y2="12"></line>
        <polyline points="12 19 5 12 12 5"></polyline>
      </svg>
      返回首頁
    </button>
    
    <h1 class="title">JWT 解碼器</h1>
    
    <div class="editor-container">
      <!-- 輸入區塊 -->
      <div class="editor-pane input-section">
        <div class="pane-header input-header">
          <span>請貼上 JWT Token</span>
          <button class="btn btn-secondary clear-btn" @click="clearAll">清除</button>
        </div>
        <textarea 
          v-model="inputToken" 
          class="code-area input-control" 
          placeholder="eyJhbGciOiJIUzI1NiIsInR..."
        ></textarea>
      </div>
      
      <!-- 輸出區塊 -->
      <div class="editor-pane result-section">
        <div class="pane-header">解碼結果</div>
        
        <div v-if="!inputToken" class="empty-state">
          等待輸入 Token...
        </div>
        
        <div v-else-if="decodedData.error" class="error-msg">
          {{ decodedData.error }}
        </div>
        
        <div v-else class="decoded-content">
          <div class="section-title">Header (標頭)</div>
          <pre class="json-block">{{ JSON.stringify(decodedData.header, null, 2) }}</pre>

          <div class="section-title">Payload (資料內容)</div>
          <pre class="json-block payload-block">{{ JSON.stringify(decodedData.payload, null, 2) }}</pre>
          
          <div v-if="decodedData.payload.iat || decodedData.payload.exp" class="time-info">
            <div class="section-title">時間資訊解析</div>
            <div class="time-item" v-if="decodedData.payload.iat">
              <strong>發行時間 (iat):</strong> {{ formatTimestamp(decodedData.payload.iat) }}
            </div>
            <div class="time-item" v-if="decodedData.payload.exp">
              <strong>過期時間 (exp):</strong> {{ formatTimestamp(decodedData.payload.exp) }}
              <span class="status-badge" :class="{ expired: (decodedData.payload.exp * 1000) < Date.now() }">
                {{ (decodedData.payload.exp * 1000) < Date.now() ? '已過期' : '有效中' }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.jwt-card {
  max-width: 1000px;
  width: 95%;
}

.editor-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  height: 550px;
  margin-top: 1.5rem;
}

@media (max-width: 768px) {
  .editor-container {
    grid-template-columns: 1fr;
    height: auto;
    min-height: 800px;
  }
}

.editor-pane {
  display: flex;
  flex-direction: column;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  overflow: hidden;
}

.pane-header {
  padding: 10px 15px;
  background: rgba(0, 0, 0, 0.3);
  font-size: 14px;
  font-weight: bold;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  color: #f8fafc;
}

.input-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.clear-btn {
  margin-top: 0;
  width: auto;
  padding: 0.2rem 0.8rem;
  font-size: 0.85rem;
}

.code-area {
  flex: 1;
  background: transparent;
  border: none;
  padding: 15px;
  color: #e2e8f0;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 14px;
  line-height: 1.5;
  resize: none;
  border-radius: 0;
  word-break: break-all;
}

.code-area:focus {
  outline: none;
  box-shadow: none;
}

.result-section {
  background: rgba(255, 255, 255, 0.05);
  overflow-y: auto;
}

.empty-state {
  padding: 2rem;
  text-align: center;
  color: #94a3b8;
}

.error-msg {
  padding: 1.5rem;
  color: #ef4444;
  font-weight: bold;
}

.decoded-content {
  padding: 15px;
}

.section-title {
  font-size: 13px;
  color: #94a3b8;
  margin-bottom: 8px;
  font-weight: bold;
  text-transform: uppercase;
}

.json-block {
  background: rgba(0, 0, 0, 0.3);
  padding: 15px;
  border-radius: 6px;
  color: #f87171; /* header redish */
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 14px;
  margin-bottom: 1.5rem;
  overflow-x: auto;
}

.payload-block {
  color: #a3e635; /* payload greenish */
}

.time-info {
  background: rgba(59, 130, 246, 0.1);
  border: 1px solid rgba(59, 130, 246, 0.3);
  padding: 15px;
  border-radius: 6px;
}

.time-item {
  margin-bottom: 8px;
  font-size: 14px;
  color: #e2e8f0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.time-item:last-child {
  margin-bottom: 0;
}

.status-badge {
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: bold;
  background: #22c55e;
  color: white;
}

.status-badge.expired {
  background: #ef4444;
}
</style>
