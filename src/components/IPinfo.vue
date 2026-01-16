<template>
    <div class="ip-container">
        <div class="ip-cards">
            <transition name="el-fade-in">
                <div v-if="ipInfo.local && ipInfo.local.country && ipInfo.local.country.code == 'CN'" class="ip-card">
                    <div class="ip-card-header">
                        <span class="ip-label">本地 IP</span>
                        <el-tag class="ip-tag">{{ ipInfo.layLocal ? ipInfo.layLocal + "ms" : "-ms" }}</el-tag>
                    </div>
                    <div class="ip-info" @click.stop="onQuery(ipInfo.local.ip)">
                        <div class="ip-address">{{ ipInfo.local.ip }}</div>
                        <div class="ip-details">{{ ipInfo.local.show.join(" / ") }}</div>
                    </div>
                </div>
            </transition>

            <transition name="el-fade-in">
                <div v-if="ipInfo.cloudflare && ipInfo.cloudflare.country && ipInfo.cloudflare.country.code != 'CN'" class="ip-card">
                    <div class="ip-card-header">
                        <span class="ip-label">Cloudflare IP</span>
                        <el-tag class="ip-tag">{{ ipInfo.layCloudflare ? ipInfo.layCloudflare + "ms" : "-ms" }}</el-tag>
                    </div>
                    <div class="ip-info" @click.stop="onQuery(ipInfo.cloudflare.ip)">
                        <div class="ip-address">{{ ipInfo.cloudflare.ip }}</div>
                        <div class="ip-details">{{ ipInfo.cloudflare.show.join(" / ") }}</div>
                    </div>
                </div>
            </transition>

            <transition name="el-fade-in">
                <div v-if="!ipInfo.local && !ipInfo.cloudflare" class="ip-card loading-card">
                    <el-skeleton :rows="3" animated />
                </div>
            </transition>
        </div>
    </div>

    <el-dialog align-center style="width: 95vw; max-width: 600px; max-height: 85vh;" v-model="queryWindow" title="IP查询">
        <div class="query-container">
            <div class="query-input-wrapper">
                <el-input v-model="ipInput" placeholder="请输入IPV4/IPV6地址" clearable autocomplete="new-password">
                    <template #append>
                        <el-button type="primary" @click="onQuery(ipInput)" :icon="Search" circle />
                    </template>
                </el-input>
            </div>

            <div v-loading="isQuerying" class="ip-result-container">
                <div class="ip-result-card">
                    <div class="result-item">
                        <span class="result-label">IP 地址</span>
                        <span class="result-value clickable" @click="copy(ipRet.ip)">{{ ipRet.ip }}</span>
                    </div>

                    <div class="result-item">
                        <span class="result-label">地址</span>
                        <span class="result-value">{{ ipRet.addr }}</span>
                    </div>

                    <div v-if="ipRet.as?.info || ipRet.type" class="result-item">
                        <span class="result-label">类型</span>
                        <span class="result-value">{{ ipRet.as?.info }} {{ ipRet.type }}</span>
                    </div>

                    <div v-if="ipRet.as" class="result-item">
                        <span class="result-label">ASN</span>
                        <span class="result-value">{{ ipRet.as?.number }}</span>
                    </div>

                    <div v-if="ipRet.country" class="result-item">
                        <span class="result-label">国家/地区</span>
                        <span class="result-value">{{ ipRet.country?.name }}({{ ipRet.country?.code }}) {{ ipRet.regions?.join(" ") }}</span>
                    </div>

                    <div v-if="ipRet.registered_country?.code != ipRet.country?.code" class="result-item">
                        <span class="result-label">注册地</span>
                        <span class="result-value">{{ ipRet.registered_country?.name }}({{ ipRet.registered_country?.code }})</span>
                    </div>

                    <div class="result-item">
                        <span class="result-label">运营商</span>
                        <span class="result-value">{{ ipRet.as?.name }}</span>
                    </div>
                </div>
            </div>
        </div>
    </el-dialog>
</template>

<script lang="ts" setup>
const props = defineProps({
    isVisible: Boolean
})
import { reactive, ref, type Ref } from 'vue'
import { ElMessage } from 'element-plus'
import { toClipboard } from '@soerenmartius/vue3-clipboard'
import { Search } from '@element-plus/icons-vue'

const queryWindow = ref(false)
const isQuerying = ref(false)
const ipInput = ref("")
const ipRet: Ref<any> = ref({})

const onQuery = async(ip:string) => {
    queryWindow.value = true
    try {
        const ipv4 = /^((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$/;
        const ipv6 = /^([\da-fA-F]{1,4}:){6}((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^::([\da-fA-F]{1,4}:){0,4}((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^([\da-fA-F]{1,4}:):([\da-fA-F]{1,4}:){0,3}((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^([\da-fA-F]{1,4}:){2}:([\da-fA-F]{1,4}:){0,2}((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^([\da-fA-F]{1,4}:){3}:([\da-fA-F]{1,4}:){0,1}((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^([\da-fA-F]{1,4}:){4}:((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^([\da-fA-F]{1,4}:){7}[\da-fA-F]{1,4}$|^:((:[\da-fA-F]{1,4}){1,6}|:)$|^[\da-fA-F]{1,4}:((:[\da-fA-F]{1,4}){1,5}|:)$|^([\da-fA-F]{1,4}:){2}((:[\da-fA-F]{1,4}){1,4}|:)$|^([\da-fA-F]{1,4}:){3}((:[\da-fA-F]{1,4}){1,3}|:)$|^([\da-fA-F]{1,4}:){4}((:[\da-fA-F]{1,4}){1,2}|:)$|^([\da-fA-F]{1,4}:){5}:([\da-fA-F]{1,4})?$|^([\da-fA-F]{1,4}:){6}:$/;
    
        if (!ipv4.test(ip) && !ipv6.test(ip)) {
            throw "请输入正确的IP地址"
        }
        isQuerying.value = true
        ipRet.value = await cachedQuery(ip)
        isQuerying.value = false
    } catch (error) {
        ElMessage.error(String(error))
    }
}

const ipInfo: {local:any, cloudflare:any,layLocal:any,layCloudflare:any} = reactive({local:null, cloudflare:null,layLocal:null,layCloudflare:null})

const copy = (ip: string) => {
    toClipboard(ip)
    ElMessage.success({
        dangerouslyUseHTMLString: true,
        message: `已复制IP地址：<br><strong>${ip}</strong>`,
    })
}

async function queryIp(ip: string) {
    const rsp = await fetch(import.meta.env.VITE_API_URL + "ip.ajax?ip=" + ip, {
        method: "GET",
        mode: "cors",
        redirect: "follow",
        referrerPolicy: "no-referrer"
    });
    let resp = await rsp.json();
    return resp['data']
}

let failure = false
async function cachedQuery(ip: string) {
    let ret = JSON.parse(localStorage.getItem("cache_ip_"+ip) || "{}")
    if (!ret.ip || new Date().getTime() / 1000 - ret.time > 60 * 60 * 24){
        try {
            if(failure) throw ""
            ret = await queryIp(ip)
        } catch (error) {
            failure = true
            throw "查询IP信息失败"
        }
        ret['time'] = new Date().getTime() / 1000
        localStorage.setItem("cache_ip_"+ip, JSON.stringify(ret))
    }
    return ret
}

async function handleIP(ip: string) {
    const info = await cachedQuery(ip)
    info.show = info.regions_short || info.regions || []
    if (info.country && info.country.code != "CN") {
        info.show = [info.country.name, ...info.show]
    }
    if (info.as) {
        info.show.push(info.as.info || info.as.name)
    }
    if (info.type) {
        info.show.push(info.type)
    }
    return info
}

(async function watchLocalIp() {
    if (props.isVisible) {
        try {
            const response = await fetch(import.meta.env.VITE_API_URL + 'ip.ajax', { referrerPolicy: 'no-referrer' });
            let resp = await response.json();
            ipInfo.local = await handleIP(resp["data"]["ip"])
        } catch (error) {
            console.log(error)
        }
    }
    setTimeout(watchLocalIp, 60000)
})();

const watchCloudflare = async(host: string) => {
    if (props.isVisible) {
        try {
            var start_timestamp = new Date().getTime();
            const response = await fetch(`https://${host}/cdn-cgi/trace`, { referrerPolicy: 'no-referrer' });
            const lay = new Date().getTime() - start_timestamp;
            let resp = await response.text();
            let ip = resp.match(/ip=([0-9a-f.:]+)/);
            if (ip) {
                ipInfo.cloudflare = await handleIP(ip[1])
                ipInfo.layCloudflare = lay
            }
        } catch (error) {
            console.log(error)
        }
    }
    setTimeout(watchCloudflare, 1000, host)
}

watchCloudflare("cp.cloudflare.com")

;(async function getCNLay() {
    if (props.isVisible) {
        try {
            var start_timestamp = new Date().getTime();
            await fetch("https://connectivitycheck.platform.hicloud.com/generate_204",
            { method: "HEAD", cache: "no-store", mode: 'no-cors', referrerPolicy: 'no-referrer' });
            ipInfo.layLocal = new Date().getTime() - start_timestamp
        } catch (error) {
            ipInfo.layLocal = 0
        }
    }
    setTimeout(getCNLay, 1000)
})();
</script>

<style scoped>
.ip-container {
    width: 100%;
}

.ip-cards {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 16px;
}

@media (max-width: 768px) {
    .ip-cards {
        grid-template-columns: 1fr;
    }
}

.ip-card {
    backdrop-filter: blur(20px);
    background: rgba(255, 255, 255, 0.7);
    border: 1px solid rgba(255, 255, 255, 0.3);
    border-radius: 20px;
    padding: 20px;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
    transition: all 0.3s ease;
    cursor: pointer;
}

.ip-card:hover {
    background: rgba(255, 255, 255, 0.8);
    box-shadow: 0 12px 40px rgba(0, 0, 0, 0.12);
    transform: translateY(-2px);
}

@media (prefers-color-scheme: dark) {
    .ip-card {
        background: rgba(26, 26, 46, 0.6);
        border-color: rgba(255, 255, 255, 0.1);
        box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
    }

    .ip-card:hover {
        background: rgba(26, 26, 46, 0.7);
        box-shadow: 0 12px 40px rgba(0, 0, 0, 0.4);
    }
}

.loading-card {
    min-height: 120px;
    display: flex;
    align-items: center;
    justify-content: center;
}

.ip-card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
    gap: 12px;
}

.ip-label {
    font-size: 14px;
    font-weight: 600;
    color: #333;
}

@media (prefers-color-scheme: dark) {
    .ip-label {
        color: #e2e8f0;
    }
}

.ip-tag {
    padding: 4px 12px;
    border-radius: 8px;
    font-size: 12px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
}

.ip-info {
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.ip-address {
    font-size: 16px;
    font-weight: 600;
    color: #333;
    word-break: break-all;
    font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
}

@media (prefers-color-scheme: dark) {
    .ip-address {
        color: #e2e8f0;
    }
}

.ip-details {
    font-size: 13px;
    color: #666;
    line-height: 1.5;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
}

@media (prefers-color-scheme: dark) {
    .ip-details {
        color: #a0aec0;
    }
}

/* 查询对话框 */
.query-container {
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.query-input-wrapper {
    width: 100%;
}

:deep(.query-input-wrapper .el-input__wrapper) {
    background: rgba(0, 0, 0, 0.05);
    border-radius: 12px;
    border: 1px solid rgba(0, 0, 0, 0.1);
}

@media (prefers-color-scheme: dark) {
    :deep(.query-input-wrapper .el-input__wrapper) {
        background: rgba(255, 255, 255, 0.05);
        border-color: rgba(255, 255, 255, 0.1);
    }
}

.ip-result-container {
    min-height: 200px;
}

.ip-result-card {
    display: flex;
    flex-direction: column;
    gap: 16px;
    padding: 16px;
    background: rgba(0, 0, 0, 0.02);
    border-radius: 12px;
    border: 1px solid rgba(0, 0, 0, 0.05);
}

@media (prefers-color-scheme: dark) {
    .ip-result-card {
        background: rgba(255, 255, 255, 0.02);
        border-color: rgba(255, 255, 255, 0.05);
    }
}

.result-item {
    display: grid;
    grid-template-columns: 80px 1fr;
    gap: 16px;
    align-items: start;
    padding-bottom: 12px;
    border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.result-item:last-child {
    border-bottom: none;
    padding-bottom: 0;
}

@media (prefers-color-scheme: dark) {
    .result-item {
        border-bottom-color: rgba(255, 255, 255, 0.05);
    }
}

.result-label {
    font-size: 12px;
    color: #999;
    font-weight: 600;
    text-transform: uppercase;
}

@media (prefers-color-scheme: dark) {
    .result-label {
        color: #718096;
    }
}

.result-value {
    font-size: 14px;
    color: #333;
    word-break: break-word;
    line-height: 1.5;
}

@media (prefers-color-scheme: dark) {
    .result-value {
        color: #e2e8f0;
    }
}

.result-value.clickable {
    cursor: pointer;
    color: #667eea;
    transition: all 0.2s ease;
    font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
}

.result-value.clickable:hover {
    opacity: 0.8;
    text-decoration: underline;
}

@media (prefers-color-scheme: dark) {
    .result-value.clickable {
        color: #a0aec0;
    }
}

@media (max-width: 480px) {
    .ip-card {
        padding: 16px;
        border-radius: 16px;
    }

    .ip-card-header {
        margin-bottom: 12px;
    }

    .ip-label {
        font-size: 13px;
    }

    .ip-address {
        font-size: 14px;
    }

    .ip-details {
        font-size: 12px;
    }

    .result-item {
        grid-template-columns: 70px 1fr;
        gap: 12px;
    }

    .result-label {
        font-size: 11px;
    }

    .result-value {
        font-size: 13px;
    }
}
</style>
