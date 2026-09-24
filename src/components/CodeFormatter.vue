<script setup>
import { ref, onMounted } from 'vue'
import beautify from 'js-beautify'

const emit = defineEmits(['back'])

const inputCode = ref(`const example=function(){console.log("Hello, World!");let arr=[1,2,3];if(arr.length>0){return arr.map(x=>x*2)}}`)
const outputCode = ref('')
const language = ref('js') // auto-detected language
const indentSize = ref(2)
const isFormatted = ref(false)

const detectLanguage = (code) => {
  const text = code.trim()
  
  // HTML / XML
  if (/(^<\/?\w+)|(<\w+[\s>])/.test(text)) {
    return 'html'
  }
  
  // CSS (looks for basic css rules and properties, ignoring common JS keywords)
  if (/[\w\-\.\#\s]+\{/m.test(text) && !/(function|const |let |var |=>)/.test(text)) {
    if (/[a-z\-]+\s*:\s*[^;{}]+;/.test(text)) {
      return 'css'
    }
  }
  
  // Default to JS / JSON
  return 'js'
}

const formatCode = () => {
  if (!inputCode.value.trim()) {
    outputCode.value = ''
    isFormatted.value = false
    return
  }

  const options = {
    indent_size: indentSize.value,
    space_in_empty_paren: true,
    preserve_newlines: true
  }

  language.value = detectLanguage(inputCode.value)

  try {
    if (language.value === 'js') {
      outputCode.value = beautify.js(inputCode.value, options)
    } else if (language.value === 'html') {
      outputCode.value = beautify.html(inputCode.value, options)
    } else if (language.value === 'css') {
      outputCode.value = beautify.css(inputCode.value, options)
    }
    isFormatted.value = true
  } catch (error) {
    outputCode.value = '排版發生錯誤：\n' + error.message
    isFormatted.value = false
  }
}

onMounted(() => {
  formatCode()
})

const copyResult = () => {
  if (!outputCode.value) return
  
  const textarea = document.createElement('textarea')
  textarea.value = outputCode.value
  textarea.setAttribute('readonly', '')
  textarea.style.position = 'fixed'
  textarea.style.left = '-9999px'
  document.body.appendChild(textarea)
  textarea.select()
  document.execCommand('copy')
  document.body.removeChild(textarea)
  
  alert('已複製！')
}

const clearAll = () => {
  inputCode.value = ''
  outputCode.value = ''
  isFormatted.value = false
}
</script>

<template>
  <div class="glass-card formatter-card">
    <button class="btn-back" @click="emit('back')">
      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <line x1="19" y1="12" x2="5" y2="12"></line>
        <polyline points="12 19 5 12 12 5"></polyline>
      </svg>
      返回首頁
    </button>
    
    <h1 class="title">程式碼排版</h1>
    
    <div class="settings-group">
      <div class="setting-item">
        <label class="setting-label">語言：</label>
        <div class="input-control language-badge">
          自動判別為 <span class="highlight">{{ language.toUpperCase() }}</span>
        </div>
      </div>
      
      <div class="setting-item">
        <label class="setting-label">縮排大小：</label>
        <select v-model="indentSize" class="input-control select-control" @change="formatCode">
          <option :value="2">2 空格</option>
          <option :value="4">4 空格</option>
        </select>
      </div>
      
      <div class="header-actions">
        <button class="btn btn-secondary action-btn" @click="clearAll">清除</button>
        <button class="btn action-btn" @click="formatCode">一鍵排版</button>
      </div>
    </div>

    <div class="editor-container">
      <div class="editor-pane">
        <div class="pane-header">輸入 (單行或壓縮程式碼)</div>
        <textarea 
          v-model="inputCode" 
          class="code-area" 
          placeholder="請貼上需要排版的程式碼..."
          @input="formatCode"
        ></textarea>
      </div>
      
      <div class="editor-pane">
        <div class="pane-header output-header">
          <span>輸出結果</span>
          <button v-if="isFormatted" class="btn btn-secondary copy-btn" @click="copyResult">複製結果</button>
        </div>
        <textarea 
          v-model="outputCode" 
          class="code-area output-area" 
          readonly
          placeholder="排版後的程式碼會顯示在這裡..."
        ></textarea>
      </div>
    </div>
  </div>
</template>

<style scoped>
.formatter-card {
  max-width: 1000px;
  width: 95%;
}

.settings-group {
  display: flex;
  gap: 1.5rem;
  margin-bottom: 1.5rem;
  flex-wrap: wrap;
  align-items: flex-end;
}

.setting-item {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  flex: 1;
  min-width: 180px;
}

.setting-label {
  font-weight: 500;
  font-size: 0.95rem;
}

.select-control {
  cursor: pointer;
}

.language-badge {
  background: rgba(59, 130, 246, 0.2);
  border-color: rgba(59, 130, 246, 0.5);
  color: #93c5fd;
  display: flex;
  align-items: center;
  cursor: default;
}

.language-badge .highlight {
  color: #fff;
  font-weight: bold;
  margin-left: 5px;
}

.header-actions {
  display: flex;
  gap: 0.75rem;
  flex: 1.5;
  min-width: 200px;
}

.action-btn {
  margin-top: 0;
  flex: 1;
}

.editor-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  height: 500px;
}

@media (max-width: 768px) {
  .editor-container {
    grid-template-columns: 1fr;
    height: 800px;
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
}

.output-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
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
}

.code-area:focus {
  outline: none;
}

.output-area {
  color: #a3e635;
}

.copy-btn {
  margin-top: 0;
  width: auto;
  padding: 0.3rem 0.8rem;
  font-size: 0.85rem;
}
</style>
