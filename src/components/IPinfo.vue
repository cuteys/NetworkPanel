<template>
  <div class="ios-card glass">
    <div class="ip-info-container">
      <transition name="el-fade-in">
        <div v-if="ipInfo.local" class="ip-row" @click="onQuery(ipInfo.local.ip)">
          <div class="ip-tag-wrapper">
            <span class="ios-badge success">{{ ipInfo.layLocal ? ipInfo.layLocal + 'ms' : '-ms' }}</span>
          </div>
          <div class="ip-text-content">
            <span class="ip-address">{{ ipInfo.local.ip }}</span>
            <span class="ip-details">{{ ipInfo.local.show.join(' ') }}</span>
          </div>
        </div>
      </transition>

      <div class="divider" v-if="ipInfo.local && ipInfo.cloudflare"></div>

      <transition name="el-fade-in">
        <div v-if="ipInfo.cloudflare" class="ip-row" @click="onQuery(ipInfo.cloudflare.ip)">
          <div class="ip-tag-wrapper">
            <span class="ios-badge warning">{{ ipInfo.layCloudflare ? ipInfo.layCloudflare + 'ms' : '-ms' }}</span>
          </div>
          <div class="ip-text-content">
            <span class="ip-address">{{ ipInfo.cloudflare.ip }}</span>
            <span class="ip-details">{{ ipInfo.cloudflare.show.join(' ') }}</span>
          </div>
        </div>
      </transition>

      <div v-if="!ipInfo.local && !ipInfo.cloudflare" class="loading-state">
        <el-icon class="is-loading"><Loading /></el-icon>
        <span>正在获取网络信息...</span>
      </div>
    </div>

    <el-dialog v-model="queryWindow" title="IP 详细信息" width="90%" class="ios-dialog-custom" append-to-body>
      <div class="search-bar">
        <el-input v-model="ipInput" class="search-input" placeholder="请输入IP地址查询" @keyup.enter="onQuery(ipInput)" />
        <el-button class="search-btn" type="primary" :icon="Search" @click="onQuery(ipInput)">
          查询
        </el-button>
      </div>
      
      <div class="ip-details-list" v-loading="isQuerying">
        <div class="detail-item" v-if="ipRet.ip" @click="copy(ipRet.ip)">
          <span class="label">IP 地址</span>
          <span class="value">{{ ipRet.ip }}</span>
        </div>
        <div class="detail-item" v-if="ipRet.addr">
          <span class="label">网段</span>
          <span class="value">{{ ipRet.addr }}</span>
        </div>
        <div class="detail-item" v-if="ipRet.country">
          <span class="label">位置</span>
          <span class="value">{{ ipRet.country.name }} {{ ipRet.regions?.join(' ') }}</span>
        </div>
        <div class="detail-item" v-if="ipRet.as">
          <span class="label">运营商</span>
          <span class="value">{{ ipRet.as.info || ipRet.as.name }}</span>
        </div>
        <div class="detail-item" v-if="ipRet.type">
          <span class="label">类型</span>
          <span class="value">{{ ipRet.type }}</span>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import { reactive, ref, type Ref, onMounted, onUnmounted } from 'vue'
import { ElMessage } from 'element-plus'
import { toClipboard } from '@soerenmartius/vue3-clipboard'
import { Search, Loading } from '@element-plus/icons-vue'

const props = defineProps({ isVisible: Boolean })

const queryWindow = ref(false)
const isQuerying = ref(false)
const ipInput = ref("")
const ipRet: Ref<any> = ref({})
const ipInfo: any = reactive({ local: null, cloudflare: null, layLocal: null, layCloudflare: null })

const onQuery = async (ip: string) => {
  if (!ip) return
  queryWindow.value = true
  isQuerying.value = true
  try {
    ipRet.value = await cachedQuery(ip)
  } catch (e) {
    ElMessage.error('查询失败')
  } finally {
    isQuerying.value = false
  }
}

const copy = (text: string) => {
  toClipboard(text)
  ElMessage.success('已复制到剪贴板')
}

async function cachedQuery(ip: string) {
  const cacheKey = `cache_ip_${ip}`
  const cached = localStorage.getItem(cacheKey)
  if (cached) {
    const data = JSON.parse(cached)
    if (Date.now() / 1000 - data.time < 86400) return data
  }
  
  const rsp = await fetch(`${import.meta.env.VITE_API_URL}ip.ajax?ip=${ip}`)
  const res = await rsp.json()
  const data = { ...res.data, time: Date.now() / 1000 }
  localStorage.setItem(cacheKey, JSON.stringify(data))
  return data
}

async function handleIP(ip: string) {
  const info = await cachedQuery(ip)
  info.show = info.regions_short || info.regions || []
  if (info.country && info.country.code !== "CN") info.show = [info.country.name, ...info.show]
  if (info.as) info.show.push(info.as.info || info.as.name)
  if (info.type) info.show.push(info.type)
  return info
}

let localTimer: any = null
let cfTimer: any = null

const fetchLocal = async () => {
  try {
    if (!ipInfo.local) {
      const rsp = await fetch(`${import.meta.env.VITE_API_URL}ip.ajax`)
      const res = await rsp.json()
      ipInfo.local = await handleIP(res.data.ip)
    }
    
    const start = Date.now()
    await fetch("https://connectivitycheck.platform.hicloud.com/generate_204", { mode: 'no-cors', cache: 'no-store' })
    ipInfo.layLocal = Date.now() - start
  } catch (e) {}
  localTimer = setTimeout(fetchLocal, 1000)
}

const fetchCF = async () => {
  try {
    const start = Date.now()
    await fetch("https://cp.cloudflare.com/generate_204", { mode: 'no-cors', cache: 'no-store' })
    ipInfo.layCloudflare = Date.now() - start
    
    if (!ipInfo.cloudflare) {
      const rsp = await fetch("https://cp.cloudflare.com/cdn-cgi/trace")
      const text = await rsp.text()
      const ipMatch = text.match(/ip=([0-9a-f.:]+)/)
      if (ipMatch) ipInfo.cloudflare = await handleIP(ipMatch[1])
    }
  } catch (e) {}
  cfTimer = setTimeout(fetchCF, 1000)
}

onMounted(() => {
  fetchLocal()
  fetchCF()
})

onUnmounted(() => {
  clearTimeout(localTimer)
  clearTimeout(cfTimer)
})
</script>

<style scoped>
.ios-card {
  border-radius: 24px;
  padding: 16px 24px;
}

.ip-info-container {
  display: flex;
  flex-direction: column;
}

.ip-row {
  display: flex;
  align-items: center;
  padding: 16px 0;
  cursor: pointer;
  transition: opacity 0.2s;
  overflow: hidden;
}

.ip-row:active {
  opacity: 0.6;
}

.ip-tag-wrapper {
  flex-shrink: 0;
  width: 75px;
  margin-right: 5px;
}

.ios-badge {
  display: inline-block;
  padding: 6px 10px;
  border-radius: 10px;
  font-size: 13px;
  font-weight: 700;
  min-width: 50px;
  text-align: center;
}

.ios-badge.success { background: rgba(52, 199, 89, 0.15); color: #248a3d; }
.ios-badge.warning { background: rgba(255, 149, 0, 0.15); color: #b26a00; }

@media (prefers-color-scheme: dark) {
  .ios-badge.success { background: rgba(48, 209, 88, 0.2); color: #30d158; }
  .ios-badge.warning { background: rgba(255, 159, 10, 0.2); color: #ff9f0a; }
}

.ip-text-content {
  display: flex;
  align-items: center;
  gap: 16px;
  flex: 1;
  min-width: 0;
  white-space: nowrap;
}

.ip-address {
  font-size: 16px;
  font-weight: 600;
  color: var(--text-main);
  flex-shrink: 0;
  font-family: 'SF Mono', 'Monaco', monospace;
}

.ip-details {
  font-size: 14px;
  color: var(--text-secondary);
  overflow: hidden;
  text-overflow: ellipsis;
  font-weight: 500;
}

.divider {
  height: 1px;
  background: rgba(0,0,0,0.05);
  margin: 0 0 0 65px;
}

.loading-state {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 24px;
  color: var(--text-secondary);
  font-size: 14px;
}

.search-bar {
  display: flex;
  gap: 12px;
  align-items: center;
  width: 100%;
}

.search-input {
  flex: 1;
}

.search-btn {
  height: 42px;
  padding: 0 20px !important;
  flex-shrink: 0;
}

.ip-details-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 20px;
}

:deep(.el-loading-mask) {
  background-color: rgba(255, 255, 255, 0.05) !important;
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  border-radius: 18px;
}

@media (prefers-color-scheme: dark) {
  :deep(.el-loading-mask) {
    background-color: rgba(0, 0, 0, 0.4) !important;
  }
}

:deep(.el-loading-spinner .path) {
  stroke: var(--ios-blue) !important;
}

.detail-item {
  background: rgba(0,0,0,0.04);
  padding: 16px 20px;
  border-radius: 18px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: background 0.2s;
}

.detail-item:active {
  background: rgba(0,0,0,0.08);
}

.detail-item .label {
  font-size: 13px;
  color: var(--text-secondary);
  font-weight: 600;
}

.detail-item .value {
  font-size: 14px;
  font-weight: 600;
  color: var(--text-main);
}

@media (prefers-color-scheme: dark) {
  .divider { background: rgba(255,255,255,0.08); }
  .detail-item { background: rgba(255,255,255,0.08); }
  .detail-item:active { background: rgba(255,255,255,0.12); }
}

@media (max-width: 600px) {
  .ip-text-content {
    flex-direction: column;
    align-items: flex-start;
    gap: 4px;
  }
  .ip-details {
    font-size: 12px;
  }
}
</style>
