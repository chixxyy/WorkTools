<script setup>
import { ref, watch, onMounted } from 'vue'
import QRCode from 'qrcode'

const emit = defineEmits(['back'])

const text = ref('https://antigravity.google.com')
const width = ref(124)
const height = ref(124)
const lockRatio = ref(true)
const qrImageUrl = ref('')
const isGenerating = ref(false)

const onWidthChange = () => {
  if (lockRatio.value) {
    height.value = width.value
  }
}

const onHeightChange = () => {
  if (lockRatio.value) {
    width.value = height.value
  }
}

watch(lockRatio, (newVal) => {
  if (newVal) {
    height.value = width.value
  }
})

const generateQR = async () => {
  if (!text.value) {
    qrImageUrl.value = ''
    return
  }
  
  isGenerating.value = true
  try {
    const maxDim = Math.max(width.value, height.value)
    
    const squareDataUrl = await QRCode.toDataURL(text.value, {
      width: maxDim,
      margin: 2,
      color: {
        dark: '#000000',
        light: '#ffffff'
      }
    })
    
    const canvas = document.createElement('canvas')
    canvas.width = width.value
    canvas.height = height.value
    const ctx = canvas.getContext('2d')
    
    const img = new Image()
    img.src = squareDataUrl
    
    await new Promise((resolve) => {
      img.onload = () => {
        ctx.drawImage(img, 0, 0, width.value, height.value)
        qrImageUrl.value = canvas.toDataURL('image/png')
        resolve()
      }
    })
    
  } catch (err) {
    console.error(err)
  } finally {
    isGenerating.value = false
  }
}

watch([text, width, height], () => {
  generateQR()
})

onMounted(() => {
  generateQR()
})

const downloadQR = (format = 'png') => {
  if (!qrImageUrl.value) return
  
  const img = new Image()
  img.src = qrImageUrl.value
  
  img.onload = () => {
    const canvas = document.createElement('canvas')
    canvas.width = width.value
    canvas.height = height.value
    const ctx = canvas.getContext('2d')
    
    // For JPEG, we need a white background (no transparency)
    if (format === 'jpeg') {
      ctx.fillStyle = '#ffffff'
      ctx.fillRect(0, 0, canvas.width, canvas.height)
    }
    
    ctx.drawImage(img, 0, 0)
    
    const mimeType = `image/${format}`
    const url = canvas.toDataURL(mimeType, 1.0)
    
    const link = document.createElement('a')
    link.href = url
    const ext = format === 'jpeg' ? 'jpg' : format
    link.download = `qrcode.${ext}`
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
  }
}
</script>

<template>
  <div class="glass-card">
    <button class="btn-back" @click="emit('back')">
      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <line x1="19" y1="12" x2="5" y2="12"></line>
        <polyline points="12 19 5 12 12 5"></polyline>
      </svg>
      返回首頁
    </button>
    <h1 class="title">QR Code 產生器</h1>
    
    <div class="input-group">
      <label for="qr-text">網址</label>
      <input 
        id="qr-text"
        v-model="text" 
        type="text" 
        class="input-control"
        placeholder="請在此輸入文字或網址..."
      />
    </div>
    
    <div class="input-group">
      <label>圖片尺寸 (px)</label>
      <div class="dimension-group">
        <div class="dimension-input">
          <label for="qr-width">寬度 (Width)</label>
          <input 
            id="qr-width"
            v-model.number="width" 
            type="number" 
            class="input-control"
            min="50"
            max="2000"
            @input="onWidthChange"
          />
        </div>
        <div class="dimension-input">
          <label for="qr-height">高度 (Height)</label>
          <input 
            id="qr-height"
            v-model.number="height" 
            type="number" 
            class="input-control"
            min="50"
            max="2000"
            @input="onHeightChange"
            :disabled="lockRatio"
          />
        </div>
      </div>
      <div class="checkbox-group">
        <input 
          id="qr-lock"
          v-model="lockRatio"
          type="checkbox"
        />
        <label for="qr-lock">保持長寬一致</label>
      </div>
    </div>
    
    <div class="qr-preview">
      <div v-if="qrImageUrl" class="image-container">
        <img :src="qrImageUrl" alt="QR Code" class="qr-image" :style="{ width: '100%', maxWidth: `${width}px`, aspectRatio: `${width}/${height}` }" />
      </div>
      <div v-else class="empty-state">
        <span class="empty-icon">✨</span>
        <p>請輸入文字以產生 QR Code</p>
      </div>
    </div>
    
    <div class="download-section">
      <label class="download-label">下載選項</label>
      <div class="download-actions">
        <button @click="downloadQR('png')" class="btn" :disabled="!qrImageUrl">
          PNG 圖檔
        </button>
        <button @click="downloadQR('jpeg')" class="btn btn-secondary" :disabled="!qrImageUrl">
          JPG 圖檔
        </button>
        <button @click="downloadQR('webp')" class="btn btn-secondary" :disabled="!qrImageUrl">
          WEBP 圖檔
        </button>
      </div>
    </div>
  </div>
</template>
