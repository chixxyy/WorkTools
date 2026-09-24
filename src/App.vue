<script setup>
import { ref } from 'vue'
import QrGenerator from './components/QrGenerator.vue'
import ImageResizer from './components/ImageResizer.vue'
import CodeFormatter from './components/CodeFormatter.vue'
import JwtDecoder from './components/JwtDecoder.vue'

const currentView = ref('home')
</script>

<template>
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
