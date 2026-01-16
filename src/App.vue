<template>
  <div class="app-container">
    <header class="header">
      <h1 class="title">ChuiYan SpeedTest</h1>
    </header>

    <main class="main-content">
      <MainUI :isVisible="isVisible" />
      <br>
      <IPinfoUI :isVisible="isVisible" />
    </main>

    <footer class="footer">
      <div class="footer-links">
        <a @click='open("https://netart.cn/")' class="footer-link">Whoami</a>
        <a @click='open("https://github.com/ljxi/NetworkPanel")' class="footer-link">源码</a>
        <a @click='open("https://github.com/ljxi/GeoCN")' class="footer-link">IP离线库</a>
      </div>
      <div class="footer-cdn">
        <span class="footer-text">
          本工具由
          <a @click='open("https://edgeone.ai/zh?from=net.netart.cn")' class="footer-link">
            <img src="https://edgeone.ai/favicon.ico" alt="Tencent Edgeone" width="15" height="15">
            Tencent Edgeone
          </a>
          提供CDN服务
        </span>
      </div>
      <div class="footer-designed">
        <span class="footer-text">Designed by ChuiYan 💖</span>
      </div>
    </footer>
  </div>
</template>

<script lang="ts" setup>
import MainUI from "./components/Main.vue"
import IPinfoUI from "./components/IPinfo.vue"
import { ref, watch } from 'vue'

const isVisible = ref(true)

const open = (url: string) => {
  window.open(url)
}

document.addEventListener("visibilitychange", function () {
  const string = document.visibilityState
  if (string === 'hidden') isVisible.value = false
  else isVisible.value = true
})
</script>

<style scoped>
.app-container {
  max-width: var(--container-max-width);
  margin: 0 auto;
  min-height: calc(100vh - 40px);
  display: flex;
  flex-direction: column;
}

.header {
  text-align: center;
  padding: 20px 0;
}

.title {
  margin: 0;
  font-size: 2rem;
  font-weight: 700;
  background: linear-gradient(135deg, var(--primary), var(--primary-light));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.footer {
  padding: 30px 0;
  text-align: center;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.footer-links {
  display: flex;
  justify-content: center;
  gap: 20px;
  flex-wrap: wrap;
}

.footer-link {
  color: var(--text-color);
  text-decoration: none;
  padding: 8px 16px;
  border-radius: var(--radius-lg);
  transition: all 0.3s ease;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 5px;
}

.footer-link:hover {
  background: var(--primary);
  color: white;
}

.footer-link img {
  vertical-align: middle;
}

.footer-text {
  color: var(--text-color);
  font-size: 14px;
  opacity: 0.8;
}

.footer-cdn {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 5px;
}

.footer-designed {
  margin-top: 10px;
}

@media (max-width: 768px) {
  .title {
    font-size: 1.5rem;
  }

  .footer-links {
    gap: 10px;
  }

  .footer-link {
    padding: 6px 12px;
    font-size: 14px;
  }

  .footer-text {
    font-size: 12px;
  }
}
</style>
