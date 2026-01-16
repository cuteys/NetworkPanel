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
  --glass-bg: rgba(255, 255, 255, 0.6);
  --glass-bg-dark: rgba(28, 28, 30, 0.7);
  --glass-stroke: rgba(255, 255, 255, 0.5);
  --glass-stroke-dark: rgba(255, 255, 255, 0.15);
  --text-main: #1c1c1e;
  --text-secondary: #3a3a3c;
  --text-placeholder: #8e8e93;
  --radius-ios: 16px;
  --radius-ios-lg: 24px;
  --page-bg: linear-gradient(135deg, #a1c4fd 0%, #c2e9fb 100%);
}

@media (prefers-color-scheme: dark) {
  :root {
    --text-main: #ffffff;
    --text-secondary: #d1d1d6;
    --text-placeholder: #636366;
    --page-bg: radial-gradient(1200px 600px at 50% -10%, rgba(18, 22, 28, 0.85), rgba(10, 12, 16, 0.95)), linear-gradient(180deg, #0b1220, #0a0d14);
    --glass-bg: var(--glass-bg-dark);
    --glass-stroke: var(--glass-stroke-dark);
  }
}

* {
  box-sizing: border-box;
  -webkit-tap-highlight-color: transparent;
}

*::-webkit-scrollbar {
  display: none !important;
  width: 0 !important;
  height: 0 !important;
}

* {
  scrollbar-width: none !important;
  -ms-overflow-style: none !important;
}

html, body {
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
  backdrop-filter: blur(25px) saturate(180%);
  -webkit-backdrop-filter: blur(25px) saturate(180%);
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
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.05);
}

.header-content {
  width: 100%;
  max-width: 800px;
  padding: 0 24px;
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;
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
  box-shadow: 0 4px 12px rgba(0, 122, 255, 0.3);
}

.ios-icon svg {
  width: 100%;
  height: 100%;
}

.ios-title {
  font-size: 20px;
  font-weight: 700;
  margin: 0;
  letter-spacing: -0.5px;
  color: var(--text-main);
}

.ios-main {
  flex: 1;
  padding: 30px 0;
}

.content-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 0 20px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.ios-footer {
  padding: 40px 0;
  text-align: center;
  color: var(--text-secondary);
  font-size: 13px;
  font-weight: 500;
  opacity: 0.8;
}

.el-button {
  border-radius: 14px !important;
  font-weight: 600 !important;
  transition: all 0.2s !important;
  border: none !important;
}

.el-button--primary {
  background-color: var(--ios-blue) !important;
}

.el-button:active {
  transform: scale(0.96);
  opacity: 0.8;
}

.el-input__wrapper {
  background-color: rgba(0, 0, 0, 0.05) !important;
  border-radius: 14px !important;
  box-shadow: none !important;
  border: 1px solid transparent !important;
  padding: 0 12px !important;
  height: 42px !important;
  transition: all 0.2s !important;
}

@media (prefers-color-scheme: dark) {
  .el-input__wrapper {
    background-color: rgba(255, 255, 255, 0.1) !important;
  }
}

.el-input__wrapper.is-focus {
  background-color: rgba(0, 0, 0, 0.02) !important;
  box-shadow: 0 0 0 1px var(--ios-blue) inset !important;
}

@media (prefers-color-scheme: dark) {
  .el-input__wrapper.is-focus {
    background-color: rgba(255, 255, 255, 0.05) !important;
  }
}

.el-input__inner::placeholder {
  color: var(--text-placeholder) !important;
}

.el-input--suffix .el-input__wrapper {
  border-radius: 14px !important;
}

.el-select .el-input__wrapper {
  border-radius: 14px !important;
}

.el-select-group__wrap {
  padding: 8px 0 !important;
}

.el-select-group__wrap:not(:last-child) {
  border-bottom: none !important;
}

.el-select-group__wrap::after {
  display: none !important;
}

.el-select-group__title {
  font-size: 12px !important;
  font-weight: 700 !important;
  color: var(--ios-blue) !important;
  padding: 4px 16px !important;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.el-select-dropdown__item {
  border-radius: 8px !important;
  margin: 0 8px !important;
  height: 36px !important;
  line-height: 36px !important;
}

.el-input-group--append .el-input__wrapper {
  border-radius: 14px !important;
}

.el-dialog {
  border-radius: 24px !important;
  overflow: hidden;
  background: var(--glass-bg) !important;
  backdrop-filter: blur(40px) saturate(180%) !important;
  -webkit-backdrop-filter: blur(40px) saturate(180%) !important;
  border: 1px solid var(--glass-stroke) !important;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.2) !important;
  max-width: 600px !important;
}

@media (max-width: 600px) {
  .el-dialog {
    width: 92% !important;
    margin-top: 15vh !important;
  }
}

.el-dialog__header {
  padding: 24px 24px 12px !important;
  margin-right: 0 !important;
}

.el-dialog__title {
  font-weight: 700 !important;
  font-size: 19px !important;
  color: var(--text-main) !important;
}

.el-dialog__body {
  padding: 12px 24px 24px !important;
}

.el-dialog__footer {
  padding: 0 24px 24px !important;
}

.ios-popper {
  border-radius: 18px !important;
  padding: 16px !important;
  background: var(--glass-bg) !important;
  backdrop-filter: blur(25px) !important;
  border: 1px solid var(--glass-stroke) !important;
  box-shadow: 0 10px 40px rgba(0,0,0,0.1) !important;
}
</style>
