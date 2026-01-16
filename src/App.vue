<template>
  <div class="app-container">
    <header class="app-header">
      <div class="header-content">
        <div class="logo-section">
          <div class="logo-icon">
            <svg t="1702892592527" class="logo-svg" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg"
              p-id="4460">
              <path
                d="M568.96 459.242667l144-189.205334A382.208 382.208 0 0 0 512 213.333333c-212.074667 0-384 171.925333-384 384a382.293333 382.293333 0 0 0 93.397333 251.008l-64.554666 55.808A467.584 467.584 0 0 1 42.666667 597.333333C42.666667 338.133333 252.8 128 512 128c93.098667 0 179.861333 27.093333 252.842667 73.856l75.882666-99.690667 67.541334 51.413334-273.28 359.04a149.333333 149.333333 0 1 1-66.048-53.376z m266.453333-69.056l54.357334-71.424A467.242667 467.242667 0 0 1 981.333333 597.333333c0 113.706667-40.64 221.226667-113.237333 305.728l-64.725333-55.616A382.272 382.272 0 0 0 896 597.333333c0-76.288-22.250667-147.370667-60.586667-207.146666zM512 661.333333a64 64 0 1 0 0-128 64 64 0 0 0 0 128z"
                fill="currentColor" p-id="4461"></path>
            </svg>
          </div>
          <span class="logo-title">ChuiYan SpeedTest</span>
        </div>
      </div>
    </header>

    <main class="app-main">
      <MainUI :isVisible="isVisible" />
      <br>
      <IPinfoUI :isVisible="isVisible" />
    </main>

    <footer class="app-footer">
      <p class="footer-text">Designed by ChuiYan 💖</p>
    </footer>
  </div>
</template>

<script lang="ts" setup>
import MainUI from "./components/Main.vue"
import IPinfoUI from "./components/IPinfo.vue"
import { ref } from 'vue'

const isVisible = ref(true)

document.addEventListener("visibilitychange", function () {
  var string = document.visibilityState
  if (string === 'hidden') isVisible.value = false
  else isVisible.value = true
})
</script>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

@media (prefers-color-scheme: dark) {
  .app-container {
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  }
}

.app-header {
  position: sticky;
  top: 0;
  z-index: 100;
  backdrop-filter: blur(20px);
  background: rgba(255, 255, 255, 0.7);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  padding: 16px 0;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}

@media (prefers-color-scheme: dark) {
  .app-header {
    background: rgba(26, 26, 46, 0.8);
    border-bottom-color: rgba(255, 255, 255, 0.1);
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  }
}

.header-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.logo-section {
  display: flex;
  align-items: center;
  gap: 12px;
}

.logo-icon {
  width: 44px;
  height: 44px;
  border-radius: 12px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
}

.logo-svg {
  width: 28px;
  height: 28px;
  color: white;
}

.logo-title {
  font-size: 20px;
  font-weight: 600;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

@media (prefers-color-scheme: dark) {
  .logo-title {
    background: linear-gradient(135deg, #a0aec0 0%, #cbd5e0 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }
}

.app-main {
  flex: 1;
  max-width: 1200px;
  width: 100%;
  margin: 0 auto;
  padding: 24px 20px;
}

.app-footer {
  padding: 24px 20px;
  text-align: center;
  backdrop-filter: blur(20px);
  background: rgba(255, 255, 255, 0.5);
  border-top: 1px solid rgba(255, 255, 255, 0.2);
}

@media (prefers-color-scheme: dark) {
  .app-footer {
    background: rgba(26, 26, 46, 0.6);
    border-top-color: rgba(255, 255, 255, 0.1);
  }
}

.footer-text {
  margin: 0;
  font-size: 14px;
  color: #666;
}

@media (prefers-color-scheme: dark) {
  .footer-text {
    color: #a0aec0;
  }
}

@media (max-width: 768px) {
  .app-header {
    padding: 12px 0;
  }

  .logo-icon {
    width: 40px;
    height: 40px;
  }

  .logo-svg {
    width: 24px;
    height: 24px;
  }

  .logo-title {
    font-size: 18px;
  }

  .app-main {
    padding: 16px 12px;
  }

  .app-footer {
    padding: 16px 12px;
  }

  .footer-text {
    font-size: 12px;
  }
}
</style>
