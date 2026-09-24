<script setup>
import { ref } from 'vue'
import QrGenerator from './components/QrGenerator.vue'
import ImageResizer from './components/ImageResizer.vue'
import CodeFormatter from './components/CodeFormatter.vue'
import JwtDecoder from './components/JwtDecoder.vue'

const currentView = ref('home')
const isDarkMode = ref(window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches)

// Initialize dark mode based on user's OS preference
if (isDarkMode.value) {
  document.documentElement.classList.add('dark')
}

const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value
  if (isDarkMode.value) {
    document.documentElement.classList.add('dark')
  } else {
    document.documentElement.classList.remove('dark')
  }
}
</script>

<template>
  <div class="theme-toggle" @click="toggleDarkMode">
    <svg v-if="!isDarkMode" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
    </svg>
    <svg v-else xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <circle cx="12" cy="12" r="5"></circle>
      <line x1="12" y1="1" x2="12" y2="3"></line>
      <line x1="12" y1="21" x2="12" y2="23"></line>
      <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
      <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
      <line x1="1" y1="12" x2="3" y2="12"></line>
      <line x1="21" y1="12" x2="23" y2="12"></line>
      <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
      <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
    </svg>
  </div>

  <div v-if="currentView === 'home'" class="home-container">
    <div class="glass-card home-card">
      <h1 class="title">工具</h1>
      
      <div class="tools-grid">
        <div class="tool-item" @click="currentView = 'qrcode'">
          <div class="tool-icon">
            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <rect x="3" y="3" width="7" height="7"></rect>
              <rect x="14" y="3" width="7" height="7"></rect>
              <rect x="14" y="14" width="7" height="7"></rect>
              <rect x="3" y="14" width="7" height="7"></rect>
            </svg>
          </div>
          <h2 class="tool-title">QR Code 產生器</h2>
        </div>
        
        <div class="tool-item" @click="currentView = 'imageresizer'">
          <div class="tool-icon">
            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
              <circle cx="8.5" cy="8.5" r="1.5"></circle>
              <polyline points="21 15 16 10 5 21"></polyline>
            </svg>
          </div>
          <h2 class="tool-title">圖片縮放轉檔</h2>
        </div>

        <div class="tool-item" @click="currentView = 'codeformatter'">
          <div class="tool-icon">
            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <polyline points="16 18 22 12 16 6"></polyline>
              <polyline points="8 6 2 12 8 18"></polyline>
            </svg>
          </div>
          <h2 class="tool-title">程式碼排版</h2>
        </div>

        <div class="tool-item" @click="currentView = 'jwtdecoder'">
          <div class="tool-icon">
            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect>
              <path d="M7 11V7a5 5 0 0 1 10 0v4"></path>
            </svg>
          </div>
          <h2 class="tool-title">JWT 解碼器</h2>
        </div>
      </div>
    </div>
  </div>

  <QrGenerator v-else-if="currentView === 'qrcode'" @back="currentView = 'home'" />
  <ImageResizer v-else-if="currentView === 'imageresizer'" @back="currentView = 'home'" />
  <CodeFormatter v-else-if="currentView === 'codeformatter'" @back="currentView = 'home'" />
  <JwtDecoder v-else-if="currentView === 'jwtdecoder'" @back="currentView = 'home'" />
</template>

<style>
.theme-toggle {
  position: fixed;
  top: 20px;
  right: 20px;
  width: 45px;
  height: 45px;
  border-radius: 50%;
  background: var(--glass-bg);
  backdrop-filter: blur(8px);
  border: 1px solid var(--glass-border);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 9999;
  color: var(--text-main);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  transition: all 0.3s ease;
}

.theme-toggle:hover {
  transform: scale(1.1);
  background: rgba(255, 255, 255, 0.4);
}

html.dark .theme-toggle:hover {
  background: rgba(0, 0, 0, 0.4);
}
</style>
