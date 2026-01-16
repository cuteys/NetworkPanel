<template>
  <el-container class="main-container">
    <el-header class="header glass">
      <div class="header-content">
        <div class="logo-title">
          <div class="icon">
            <svg t="1702892592527" class="icon-svg" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg">
              <path
                d="M568.96 459.242667l144-189.205334A382.208 382.208 0 0 0 512 213.333333c-212.074667 0-384 171.925333-384 384a382.293333 382.293333 0 0 0 93.397333 251.008l-64.554666 55.808A467.584 467.584 0 0 1 42.666667 597.333333C42.666667 338.133333 252.8 128 512 128c93.098667 0 179.861333 27.093333 252.842667 73.856l75.882666-99.690667 67.541334 51.413334-273.28 359.04a149.333333 149.333333 0 1 1-66.048-53.376z m266.453333-69.056l54.357334-71.424A467.242667 467.242667 0 0 1 981.333333 597.333333c0 113.706667-40.64 221.226667-113.237333 305.728l-64.725333-55.616A382.272 382.272 0 0 0 896 597.333333c0-76.288-22.250667-147.370667-60.586667-207.146666zM512 661.333333a64 64 0 1 0 0-128 64 64 0 0 0 0 128z"
                fill="#ffffff"></path>
            </svg>
          </div>
          <span class="title"> ChuiYan SpeedTest </span>
        </div>
      </div>
    </el-header>
    <el-main class="content-wrapper">
      <div class="description-card glass">
        <p>专业、精准、极简的网络测速工具</p>
      </div>
      <MainUI :isVisible="isVisible" />
      <br>
      <IPinfoUI :isVisible="isVisible" />
    </el-main>
    <el-footer class="footer">
      <div class="footer-content">
        <span>Designed by ChuiYan 💖</span>
      </div>
    </el-footer>
  </el-container>
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

<style>
:root {
  --bg-blur: 20px;
  --glass-bg: rgba(255, 255, 255, 0.4);
  --glass-stroke: rgba(255, 255, 255, 0.5);
  --text-color: #1f2937;
  --primary: #007aff;
  --primary-light: #5ac8fa;
  --page-bg: linear-gradient(135deg, #a1c4fd 0%, #c2e9fb 100%);
  --radius-lg: 20px;
  --radius-xl: 30px;
}

@media (prefers-color-scheme: dark) {
  :root {
    --page-bg: radial-gradient(
        1200px 600px at 50% -10%,
        rgba(18, 22, 28, 0.85),
        rgba(10, 12, 16, 0.95)
      ),
      linear-gradient(180deg, #0b1220, #0a0d14);
    --glass-bg: rgba(20, 24, 28, 0.5);
    --glass-stroke: rgba(255, 255, 255, 0.15);
    --primary: #0a84ff;
    --primary-light: #64d2ff;
    --text-color: #e5e7eb;
  }
}

body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "Helvetica Neue", Helvetica, Arial, sans-serif;
  background: var(--page-bg);
  color: var(--text-color);
  min-height: 100vh;
}

.glass {
  background: var(--glass-bg);
  -webkit-backdrop-filter: blur(var(--bg-blur)) saturate(1.2);
  backdrop-filter: blur(var(--bg-blur)) saturate(1.2);
  border: 1px solid var(--glass-stroke);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.1);
}

.main-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  height: 70px !important;
  display: flex;
  align-items: center;
  justify-content: center;
  position: sticky;
  top: 0;
  z-index: 100;
  margin-bottom: 20px;
}

.header-content {
  width: 100%;
  max-width: 1000px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
}

.logo-title {
  display: flex;
  align-items: center;
  gap: 12px;
}

.icon {
  width: 40px;
  height: 40px;
  background-color: var(--primary);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.icon-svg {
  width: 28px;
  height: 28px;
}

.title {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--primary);
}

.content-wrapper {
  flex: 1;
  max-width: 1000px;
  margin: 0 auto;
  width: 100%;
  padding: 0 20px 40px 20px;
  box-sizing: border-box;
}

.description-card {
  padding: 15px 25px;
  border-radius: var(--radius-lg);
  margin-bottom: 25px;
  text-align: center;
}

.description-card p {
  margin: 0;
  font-size: 1.1rem;
  font-weight: 500;
  opacity: 0.9;
}

.footer {
  height: 60px !important;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: auto;
}

.footer-content {
  font-size: 0.9rem;
  opacity: 0.8;
  font-weight: 500;
}

/* Element Plus Overrides */
.el-card {
  background: var(--glass-bg) !important;
  backdrop-filter: blur(var(--bg-blur)) saturate(1.2) !important;
  border: 1px solid var(--glass-stroke) !important;
  border-radius: var(--radius-lg) !important;
  color: var(--text-color) !important;
}
</style>
