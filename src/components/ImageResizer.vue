<script setup>
import { ref, watch } from 'vue'

const emit = defineEmits(['back'])

const imageSrc = ref('')
const fileName = ref('')
const originalWidth = ref(0)
const originalHeight = ref(0)
const width = ref(0)
const height = ref(0)
const lockRatio = ref(true)

const onFileChange = (e) => {
  const file = e.target.files[0]
  if (!file) return
  
  fileName.value = file.name

  const reader = new FileReader()
  reader.onload = (event) => {
    const img = new Image()
    img.onload = () => {
      originalWidth.value = img.width
      originalHeight.value = img.height
      width.value = img.width
      height.value = img.height
      imageSrc.value = event.target.result
    }
    img.src = event.target.result
  }
  reader.readAsDataURL(file)
}

const onWidthChange = () => {
  if (lockRatio.value && originalWidth.value > 0) {
    const ratio = originalHeight.value / originalWidth.value
    height.value = Math.round(width.value * ratio)
  }
}

const onHeightChange = () => {
  if (lockRatio.value && originalHeight.value > 0) {
    const ratio = originalWidth.value / originalHeight.value
    width.value = Math.round(height.value * ratio)
  }
}

watch(lockRatio, (newVal) => {
  if (newVal && originalWidth.value > 0) {
    const ratio = originalHeight.value / originalWidth.value
    height.value = Math.round(width.value * ratio)
  }
})

const downloadImage = (format = 'png') => {
  if (!imageSrc.value) return
  
  const img = new Image()
  img.src = imageSrc.value
  
  img.onload = () => {
    const canvas = document.createElement('canvas')
    canvas.width = width.value
    canvas.height = height.value
    const ctx = canvas.getContext('2d')
    
    // For JPEG, fill with white background
    if (format === 'jpeg') {
      ctx.fillStyle = '#ffffff'
      ctx.fillRect(0, 0, canvas.width, canvas.height)
    }
    
    ctx.drawImage(img, 0, 0, width.value, height.value)
    
    const mimeType = `image/${format}`
    const url = canvas.toDataURL(mimeType, 1.0)
    
    const link = document.createElement('a')
    link.href = url
    const ext = format === 'jpeg' ? 'jpg' : format
    link.download = `resized_image.${ext}`
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
    <h1 class="title">圖片縮放轉檔</h1>
    
    <div class="input-group">
      <label>上傳圖片</label>
      <div class="file-upload-wrapper">
        <input 
          id="img-upload"
          type="file" 
          accept="image/*"
          class="file-input-hidden"
          @change="onFileChange"
        />
        <label for="img-upload" class="btn btn-upload">
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
            <polyline points="17 8 12 3 7 8"></polyline>
            <line x1="12" y1="3" x2="12" y2="15"></line>
          </svg>
          點擊選擇圖片
        </label>
        <div v-if="fileName" class="file-name">{{ fileName }}</div>
      </div>
    </div>
    
    <div v-if="imageSrc" class="input-group">
      <label>調整尺寸 (px) - 原圖: {{ originalWidth }} x {{ originalHeight }}</label>
      <div class="dimension-group">
        <div class="dimension-input">
          <label for="img-width">寬度 (Width)</label>
          <input 
            id="img-width"
            v-model.number="width" 
            type="number" 
            class="input-control"
            min="1"
            @input="onWidthChange"
          />
        </div>
        <div class="dimension-input">
          <label for="img-height">高度 (Height)</label>
          <input 
            id="img-height"
            v-model.number="height" 
            type="number" 
            class="input-control"
            min="1"
            @input="onHeightChange"
            :disabled="lockRatio"
          />
        </div>
      </div>
      <div class="checkbox-group">
        <input 
          id="img-lock"
          v-model="lockRatio"
          type="checkbox"
        />
        <label for="img-lock">保持長寬原始比例</label>
      </div>
    </div>
    
    <div class="qr-preview">
      <div v-if="imageSrc" class="image-container">
        <!-- Preview image constrained visually -->
        <img :src="imageSrc" alt="Preview" class="qr-image" style="max-width: 100%; max-height: 300px; object-fit: contain;" />
      </div>
      <div v-else class="empty-state">
        <span class="empty-icon">🖼️</span>
        <p>請先上傳一張圖片</p>
      </div>
    </div>
    
    <div class="download-section">
      <label class="download-label">下載選項</label>
      <div class="download-actions">
        <button @click="downloadImage('png')" class="btn" :disabled="!imageSrc">
          PNG 圖檔
        </button>
        <button @click="downloadImage('jpeg')" class="btn btn-secondary" :disabled="!imageSrc">
          JPG 圖檔
        </button>
        <button @click="downloadImage('webp')" class="btn btn-secondary" :disabled="!imageSrc">
          WEBP 圖檔
        </button>
      </div>
    </div>
  </div>
</template>
