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

    <el-dialog v-model="queryWindow" title="IP 详细信息" width="90%" class="ios-dialog">
      <div class="query-input-group">
        <el-input v-model="ipInput" placeholder="输入 IP 地址">
          <template #append>
            <el-button :icon="Search" @click="onQuery(ipInput)" />
          </template>
        </el-input>
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
import { reactive, ref, type Ref } from 'vue'
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

// Initial Data Fetching
const fetchLocal = async () => {
  try {
    const rsp = await fetch(`${import.meta.env.VITE_API_URL}ip.ajax`)
    const res = await rsp.json()
    ipInfo.local = await handleIP(res.data.ip)
    
    const start = Date.now()
    await fetch("https://connectivitycheck.platform.hicloud.com/generate_204", { mode: 'no-cors' })
    ipInfo.layLocal = Date.now() - start
  } catch (e) {}
  setTimeout(fetchLocal, 30000)
}

const fetchCF = async () => {
  try {
    const start = Date.now()
    const rsp = await fetch("https://cp.cloudflare.com/cdn-cgi/trace")
    const text = await rsp.text()
    ipInfo.layCloudflare = Date.now() - start
    const ipMatch = text.match(/ip=([0-9a-f.:]+)/)
    if (ipMatch) ipInfo.cloudflare = await handleIP(ipMatch[1])
  } catch (e) {}
  setTimeout(fetchCF, 30000)
}

fetchLocal()
fetchCF()
</script>

<style scoped>
.ios-card {
  border-radius: var(--radius-ios-lg);
  padding: 12px 16px;
}

.ip-info-container {
  display: flex;
  flex-direction: column;
}

.ip-row {
  display: flex;
  align-items: center;
  padding: 12px 0;
  cursor: pointer;
  transition: opacity 0.2s;
  overflow: hidden;
}

.ip-row:active {
  opacity: 0.6;
}

.ip-tag-wrapper {
  flex-shrink: 0;
  width: 60px;
}

.ios-badge {
  display: inline-block;
  padding: 4px 8px;
  border-radius: 8px;
  font-size: 12px;
  font-weight: 600;
  min-width: 45px;
  text-align: center;
}

.ios-badge.success { background: rgba(52, 199, 89, 0.15); color: #248a3d; }
.ios-badge.warning { background: rgba(255, 149, 0, 0.15); color: #b26a00; }

.ip-text-content {
  display: flex;
  align-items: center;
  gap: 12px;
  flex: 1;
  min-width: 0;
  white-space: nowrap;
}

.ip-address {
  font-size: 15px;
  font-weight: 600;
  color: var(--text-main);
  flex-shrink: 0;
}

.ip-details {
  font-size: 14px;
  color: var(--text-secondary);
  overflow: hidden;
  text-overflow: ellipsis;
}

.divider {
  height: 1px;
  background: rgba(0,0,0,0.05);
  margin: 0 0 0 60px;
}

.loading-state {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 20px;
  color: var(--text-secondary);
  font-size: 14px;
}

.ip-details-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 20px;
}

.detail-item {
  background: rgba(0,0,0,0.03);
  padding: 12px 16px;
  border-radius: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.detail-item .label {
  font-size: 13px;
  color: var(--text-secondary);
}

.detail-item .value {
  font-size: 14px;
  font-weight: 600;
}

@media (prefers-color-scheme: dark) {
  .divider { background: rgba(255,255,255,0.1); }
  .detail-item { background: rgba(255,255,255,0.05); }
  .ios-badge.success { background: rgba(48, 209, 88, 0.2); color: #30d158; }
  .ios-badge.warning { background: rgba(255, 159, 10, 0.2); color: #ff9f0a; }
}

/* Mobile adjustments */
@media (max-width: 600px) {
  .ip-text-content {
    flex-direction: column;
    align-items: flex-start;
    gap: 2px;
  }
  .ip-details {
    font-size: 12px;
  }
}
</style>
