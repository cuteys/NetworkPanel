<template>
  <div class="app-container">
    <header class="ios-header glass">
      <div class="header-content">
        <div class="brand">
          <div class="ios-icon">
            <svg viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg">
              <path d="M568.96 459.242667l144-189.205334A382.208 382.208 0 0 0 512 213.333333c-212.074667 0-384 171.925333-384 384a382.293333 382.293333 0 0 0 93.397333 251.008l-64.554666 55.808A467.584 467.584 0 0 1 42.666667 597.333333C42.666667 338.133333 252.8 128 512 128c93.098667 0 179.861333 27.093333 252.842667 73.856l75.882666-99.690667 67.541334 51.413334-273.28 359.04a149.333333 149.333333 0 1 1-66.048-53.376z m266.453333-69.056l54.357334-71.424A467.242667 467.242667 0 0 1 981.333333 597.333333c0 113.706667-40.64 221.226667-113.237333 305.728l-64.725333-55.616A382.272 382.272 0 0 0 896 597.333333c0-76.288-22.250667-147.370667-60.586667-207.146666zM512 661.333333a64 64 0 1 0 0-128 64 64 0 0 0 0 128z" fill="#ffffff"></path>
            </svg>
          </div>
          <h1 class="ios-title">ChuiYan SpeedTest</h1>
        </div>
      </div>
    </header>

    <main class="ios-main">
      <div class="content-container">
        <MainUI :isVisible="isVisible" />
        <IPinfoUI :isVisible="isVisible" />
      </div>
    </main>

    <footer class="ios-footer">
      <p>Designed by ChuiYan 💖</p>
    </footer>
  </div>
</template>

<script lang="ts" setup>
import MainUI from "./components/Main.vue"
import IPinfoUI from "./components/IPinfo.vue"
import { ref, onMounted, onUnmounted } from 'vue'

const isVisible = ref(true)

const handleVisibilityChange = () => {
  isVisible.value = document.visibilityState !== 'hidden'
}

onMounted(() => {
  document.addEventListener("visibilitychange", handleVisibilityChange)
})

onUnmounted(() => {
  document.removeEventListener("visibilitychange", handleVisibilityChange)
})
</script>

<style>
:root {
  --ios-blue: #007aff;
  --ios-bg-light: #f2f2f7;
  --ios-bg-dark: #000000;
  --glass-bg: rgba(255, 255, 255, 0.7);
  --glass-bg-dark: rgba(28, 28, 30, 0.7);
  --glass-stroke: rgba(255, 255, 255, 0.3);
  --text-main: #000000;
  --text-secondary: #8e8e93;
  --radius-ios: 12px;
  --radius-ios-lg: 20px;
  --page-bg: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

@media (prefers-color-scheme: dark) {
  :root {
    --text-main: #ffffff;
    --text-secondary: #98989d;
    --page-bg: radial-gradient(circle at top, #1c1c1e 0%, #000000 100%);
    --glass-bg: var(--glass-bg-dark);
  }
}

* {
  box-sizing: border-box;
  -webkit-tap-highlight-color: transparent;
}

body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, "SF Pro Text", "SF Pro Icons", "Helvetica Neue", Helvetica, Arial, sans-serif;
  background: var(--page-bg);
  color: var(--text-main);
  min-height: 100vh;
  overflow-x: hidden;
}

.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.glass {
  background: var(--glass-bg);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border: 1px solid var(--glass-stroke);
}

.ios-header {
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: sticky;
  top: 0;
  z-index: 1000;
  box-shadow: 0 1px 0 rgba(0,0,0,0.05);
}

.header-content {
  width: 100%;
  max-width: 800px;
  padding: 0 20px;
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
}

.ios-icon {
  width: 32px;
  height: 32px;
  background: var(--ios-blue);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5px;
}

.ios-icon svg {
  width: 100%;
  height: 100%;
}

.ios-title {
  font-size: 20px;
  font-weight: 600;
  margin: 0;
  letter-spacing: -0.5px;
}

.ios-main {
  flex: 1;
  padding: 20px 0;
}

.content-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 0 16px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.ios-footer {
  padding: 30px 0;
  text-align: center;
  color: var(--text-secondary);
  font-size: 13px;
}

/* iOS Component Overrides */
.el-button.is-round {
  border-radius: 20px !important;
}

.el-select .el-input__inner {
  border-radius: 10px !important;
}

.el-dialog {
  border-radius: 14px !important;
  overflow: hidden;
}

/* Hide scrollbar */
::-webkit-scrollbar {
  display: none;
}
</style>
