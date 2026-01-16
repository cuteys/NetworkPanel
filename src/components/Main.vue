<template>
  <div class="main-container">
    <!-- 测速地址卡片 -->
    <div class="ios-card">
      <div class="card-header">
        <span class="card-title">测速地址</span>
        <div class="card-actions">
          <button class="icon-btn" @click="copyUrl" title="复制链接">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"></path>
              <rect x="8" y="2" width="8" height="4" rx="1" ry="1"></rect>
            </svg>
          </button>
          <button class="icon-btn" @click="EditTableVisible = true" title="编辑地址">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
              <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
            </svg>
          </button>
        </div>
      </div>
      <div class="ios-select-wrapper">
        <el-select v-model="runUrl" class="ios-select">
          <el-option-group v-for="group in nodes" :key="group.label" :label="group.label">
            <el-option v-for="item in group.options" :key="item.value" :label="item.label" :value="item.value" />
          </el-option-group>
        </el-select>
      </div>
    </div>

    <!-- 线程数卡片 -->
    <div class="ios-card">
      <div class="card-header">
        <span class="card-title">线程数</span>
        <span class="card-value">{{ threadNum }}</span>
      </div>
      <div class="ios-slider-wrapper">
        <el-slider :show-tooltip="false" :min="1" :max="64" v-model="threadNum" class="ios-slider" />
      </div>
    </div>

    <!-- 设置卡片 -->
    <div class="ios-card">
      <div class="settings-grid">
        <div class="setting-item">
          <span class="setting-label">保持后台运行</span>
          <el-switch v-model="runBackground" class="ios-switch" />
        </div>
        <div class="setting-item">
          <span class="setting-label">自动运行</span>
          <el-switch v-model="autoStart" class="ios-switch" />
        </div>
      </div>
    </div>

    <!-- 数据展示卡片 -->
    <div class="ios-card">
      <div class="data-grid">
        <div class="data-item">
          <div class="data-header">
            <span class="data-label">总流量</span>
            <span v-if="state.maxUse" class="data-limit">/ {{ formatter(state.maxUse, 0, [0, 0, 0, 0, 0, 0]) }}</span>
            <button class="icon-btn-small" @click="EditMaxVisible = true">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
                <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
              </svg>
            </button>
          </div>
          <div class="data-value">{{ state.show.allUsed }}</div>
        </div>

        <div class="data-item">
          <div class="data-header">
            <span class="data-label">{{ isRunning ? '实时速度' : '平均速度' }}</span>
            <el-popover placement="top-start" title="用量预测" :width="150" trigger="click">
              <template #reference>
                <button class="icon-btn-small">
                  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <circle cx="12" cy="12" r="10"></circle>
                    <polyline points="12 6 12 12 16 14"></polyline>
                  </svg>
                </button>
              </template>
              <div class="predict-grid">
                <div class="predict-item">
                  <span class="predict-label">每分钟</span>
                  <span class="predict-value">{{ state.predict.min }}</span>
                </div>
                <div class="predict-item">
                  <span class="predict-label">每小时</span>
                  <span class="predict-value">{{ state.predict.hour }}</span>
                </div>
                <div class="predict-item">
                  <span class="predict-label">每天</span>
                  <span class="predict-value">{{ state.predict.day }}</span>
                </div>
                <div class="predict-item">
                  <span class="predict-label">每月</span>
                  <span class="predict-value">{{ state.predict.mon }}</span>
                </div>
              </div>
            </el-popover>
          </div>
          <div class="data-value speed-value">{{ state.show.speed }}</div>
        </div>

        <div class="data-item">
          <div class="data-header">
            <span class="data-label">带宽</span>
            <span v-if="state.maxSpeed" class="data-limit">/ {{ formatter(state.maxSpeed, 2, [0, 0, 0, 0, 0, 0]) }}</span>
            <button class="icon-btn-small" @click="EditSpeedVisible = true">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
                <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
              </svg>
            </button>
          </div>
          <div class="data-value">{{ state.show.speedBit }}</div>
        </div>
      </div>
    </div>

    <!-- 控制按钮卡片 -->
    <div class="ios-card control-card">
      <div class="control-buttons">
        <button class="play-btn" v-if="!isRunning && !state.isChecking" @click="tryStart" :disabled="state.isChecking">
          <svg viewBox="0 0 24 24" fill="currentColor">
            <polygon points="5 3 19 12 5 21 5 3"></polygon>
          </svg>
        </button>
        <button class="play-btn loading" v-if="state.isChecking">
          <svg class="spinner" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="10"></circle>
          </svg>
        </button>
        <button class="play-btn pause" v-if="isRunning && !state.isChecking" @click="isRunning = false">
          <svg viewBox="0 0 24 24" fill="currentColor">
            <rect x="6" y="4" width="4" height="16"></rect>
            <rect x="14" y="4" width="4" height="16"></rect>
          </svg>
        </button>

        <div class="control-icons">
          <button class="control-icon-btn" @click="showMark.show = true" title="标记">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M3 12a9 9 0 1 0 18 0 9 9 0 0 0-18 0"></path>
              <path d="M12 7v5l3 2"></path>
            </svg>
          </button>
          <button class="control-icon-btn" @click="isFullScreen = true" title="全屏">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3"></path>
            </svg>
          </button>
          <button class="control-icon-btn" @click="chartShow = !chartShow" title="图表">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="12 3 20 7.5 20 16.5 12 21 4 16.5 4 7.5 12 3"></polyline>
              <polyline points="12 12 20 7.5"></polyline>
              <polyline points="12 12 12 21"></polyline>
              <polyline points="12 12 4 7.5"></polyline>
            </svg>
          </button>
        </div>
      </div>
      <div v-show="chartShow" ref="chartContainer" class="chart-container"></div>
    </div>

    <!-- 对话框 -->
    <el-dialog style="width: 90%; max-width: 700px;" v-model="EditTableVisible" title="自定义地址">
      <el-table v-if="customNodes.length" :data="customNodes" style="width: 100%" max-height="300">
        <el-table-column prop="label" label="名称" width="100" />
        <el-table-column prop="value" label="URL" />
        <el-table-column fixed="right" label="" width="50">
          <template #default="scope">
            <el-button type="danger" link @click.prevent="customNodes.splice(scope.$index, 1)" />
          </template>
        </el-table-column>
      </el-table>
      <el-empty v-else description="没有自定义地址" />
      <el-button class="mt-4" style="width: 100%" @click="addTableVisible = true;">添加地址</el-button>
    </el-dialog>

    <el-dialog style="width: 90%; max-width: 700px;" v-model="addTableVisible" title="添加链接">
      <el-form :model="addForm">
        <el-form-item label="名称:" label-width="50px">
          <el-input v-model="addForm.label" autocomplete="off" />
        </el-form-item>
        <el-form-item label="url:" label-width="50px">
          <el-input v-model="addForm.value" autocomplete="off">
            <template #suffix>
              <el-icon v-if="urlParser(addForm.value)">
                <svg viewBox="0 0 24 24" fill="currentColor">
                  <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"></path>
                </svg>
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
      </el-form>
      <el-alert title="注意：" type="warning">
        在浏览器工作的程序受到浏览器安全策略的限制
        <br>
        以下情况你将无法正常使用链接
        <br>
        1.你使用https协议打开的本站，但是url是http协议
        <br>
        2.目标服务器返回的Access-Control-Allow-Origin响应头没有允许本站
        <br>
        具体细节请在报错后查看控制台
      </el-alert>
      <el-alert title="免责声明：" type="error">
        请勿用于非法用途，使用本功能造成的一切后果由用户承担
      </el-alert>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="addTableVisible = false">取消</el-button>
          <el-button type="primary" :disabled="!urlParser(addForm.value) || !addForm.label || addForm.checking"
            @click="addNode()">确认
            <el-icon v-if="addForm.checking" class="is-loading">
              <svg class="spinner" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <circle cx="12" cy="12" r="10"></circle>
              </svg>
            </el-icon>
          </el-button>
        </span>
      </template>
    </el-dialog>

    <el-dialog style="width: 90%; max-width: 300px;" v-model="EditMaxVisible" title="设置上限自动停止">
      <el-form>
        <div class="mt-4">
          <el-input type="number" min="1" v-model="maxUseInput.num" autocomplete="off" placeholder="留空则无上限"
            class="input-with-select">
            <template #append>
              <el-select v-model="maxUseInput.type" placeholder="Select" style="width: 65px">
                <el-option label="MB" value="MB" />
                <el-option label="GB" value="GB" />
                <el-option label="TB" value="TB" />
              </el-select>
            </template>
          </el-input>
        </div>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="EditMaxVisible = false">取消</el-button>
          <el-button type="primary" @click="editMaxUse()">
            确定
          </el-button>
        </span>
      </template>
    </el-dialog>

    <el-dialog style="width: 90%; max-width: 350px;" v-model="EditSpeedVisible" title="设置带宽上限">
      <el-form>
        <div class="mt-4">
          <el-input type="number" min="1" v-model="maxSpeedInput.num" autocomplete="off" placeholder="留空则无上限"
            class="input-with-select">
            <template #append>
              <el-select v-model="maxSpeedInput.type" placeholder="Select" style="width: 80px">
                <el-option label="Mbps" value="Mbps" />
                <el-option label="Gbps" value="Gbps" />
              </el-select>
            </template>
          </el-input>
          <br><br>
          <el-alert title="注意：" type="warning">
            浏览器会使用缓存策略<br>只能限制平均速度，无法限制峰值速度!<br>部分链接无法限速，请使用其它限速方法
          </el-alert>
        </div>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="EditSpeedVisible = false">取消</el-button>
          <el-button type="primary" @click="editSpeedUse()">
            确定
          </el-button>
        </span>
      </template>
    </el-dialog>

    <MarkUI :show="showMark" :loginInfo="loginInfo" />
    <audio v-if="isMobile && !isIOS && !isMiuiBrowser && runBackground" @canplay="() => { if (isRunning) audioDom.play() }"
      @pause="() => { if (runBackground) isRunning = false }" @play="isRunning = true" controls loop ref="audioDom"
      style="display:none">
      <source :src="andoridSound" type="audio/mpeg">
    </audio>
    <audio v-if="isIOS && runBackground" @canplay="() => { if (isRunning) audioDom.play() }"
      @pause="() => { if (runBackground) isRunning = false }" @play="isRunning = true" controls loop ref="audioDom"
      style="display:none">
      <source :src="iosSound" type="audio/mpeg">
    </audio>
    <FullScreenUI v-model="isFullScreen" :isRunning="isRunning" :state="state" />
  </div>
</template>

<script lang="ts" setup>
import type { EChartsType } from "echarts";
import iosSound from "../assets/ios.mp3";
import andoridSound from "../assets/android.mp3";
const props = defineProps({
  isVisible: Boolean
})
import { ElMessage } from 'element-plus'
import nodesJson from "../assets/nodes.json"
import { ref, watch, watchEffect, type Ref, reactive } from 'vue'
import { toClipboard } from '@soerenmartius/vue3-clipboard'
import MarkUI from './Mark.vue'
import FullScreenUI from './FullScreen.vue'

const showMark = ref({ show: false })
const customNodes = reactive(localStorage.customNodes ? JSON.parse(localStorage.customNodes) : [])
const OnlineNodes: {
  label: string;
  options: {
    value: string;
    label: string;
  }[];
}[] = []
for(let groupName in nodesJson) {
  const group=nodesJson[groupName as keyof typeof nodesJson]
  const temp: {
    label: string;
    options: {
      value: string;
      label: string;
    }[];
  }={"label":groupName,options:[]}
  for(let node in group) {
    temp.options.push({"value":group[node as keyof typeof group],"label":node})
  }
  OnlineNodes.push(temp)
}
const nodes: Ref<{
  label: string;
  options: {
    value: string;
    label: string;
  }[];
}[]> = ref(OnlineNodes)
if (customNodes.length) {
  nodes.value = [{ "label": "自定义", "options": customNodes}].concat(OnlineNodes)
}
watch(customNodes, async (newState, oldState) => {
  if (customNodes.length) {
    nodes.value = [{ "label": "自定义", "options": customNodes }].concat(OnlineNodes)
  } else nodes.value = OnlineNodes
  localStorage.customNodes = JSON.stringify(newState)
}, { deep: true })

const state = reactive({
  show: {
    allUsed: '-',
    speed: '-',
    speedBit: '-'
  },
  predict: {
    min: '-',
    hour: '-',
    day: '-',
    mon: '-',
  },
  isChecking: false,
  bytesUsed: 0,
  logged: 0,
  lastLogTime: 0,
  recordUse: 0,
  recordTime: 0,
  startUse: 0,
  startTime: 0,
  maxUse: localStorage.maxUse ? Number(localStorage.maxUse) : 0,
  maxSpeed: localStorage.maxSpeed ? Number(localStorage.maxSpeed) : 0,
})
const isRunning = ref(false)
const isFullScreen = ref(false)
const loginInfo = reactive({ AccessToken: localStorage.AccessToken ? localStorage.AccessToken : "" })
const chartShow = ref(localStorage.chartShow ? localStorage.chartShow === 'true' : false)
const threadNum = ref(localStorage.threadNum ? Number(localStorage.threadNum) : 8)
const runBackground = ref(localStorage.runBackground ? localStorage.runBackground === 'true' : false)
const autoStart = ref(localStorage.autoStart ? localStorage.autoStart === 'true' : false)
const runUrl = ref(localStorage.url ? localStorage.url : nodes.value[0].options[0].value)

var tasks: Array<number> = []

import { onMounted, onUnmounted } from 'vue';
import * as echarts from 'echarts';

onMounted(() => {
  autoStart.value&&tryStart();
})

const tryStart = async () => {
  if(runUrl.value.startsWith("NetworkPanelApi://")) {
    isRunning.value = true
    return
  }
  state.isChecking = true
  const urlStatus = await checkUrl(runUrl.value)
  state.isChecking = false
  if (!urlStatus.status) {
    ElMessage.error({
      dangerouslyUseHTMLString: true,
      message: urlStatus.info
    })
  } else {
    isRunning.value = true
  }
}
const block_list=["ljxnet.cn","netart.cn",".gov.cn"]
const checkUrl = async (url: string) => {
  var status = true
  let info = ''
  try {
    let structUrl = new URL(url)
    if (block_list.some((i)=>structUrl.host.endsWith(i))) {
      throw '你不对劲，我要拿小本本把你记下来然后交给警察蜀黍！'
    }

    const controller = new AbortController();
    const id = setTimeout(() => controller.abort(), 5000);
    const response = await fetch(url, { cache: "no-store", mode: 'cors', referrerPolicy: 'no-referrer' ,signal: controller.signal})
    if (response.status == 404) throw "资源响应异常" + response.status
    if (!response.body) throw "资源响应异常 Nobody"
    const reader = response.body.getReader();
    const { value, done } = await reader.read();
    if (!value || value.length <= 0) throw "资源响应异常 Nobody";
    reader.cancel()
  } catch (err) {
    status = false
    if (err instanceof Error) info = err.message
    else info = String(err)
  }
  return {
    status: status,
    info: info,
  }
}

let solvedRunUrl = ''
async function apiSolver(){
  if(!runUrl.value.startsWith("NetworkPanelApi://")){
    solvedRunUrl = runUrl.value
    return
  }
  let host=runUrl.value.split("NetworkPanelApi://")[1]
  let resp:any = await fetch(import.meta.env.VITE_API_URL+"url.ajax?"+new URLSearchParams({host:host,cache:window.location.host}), {
      mode: "cors",
      redirect: "follow",
      referrerPolicy: "no-referrer"
    });
  resp = await resp.json()
  if(resp['status']!=0){
    isRunning.value=false;
    return
  }
  solvedRunUrl = resp['url']
}
watch(isRunning, async (newState, oldState) => {
  clearChart()
  if (newState) {
    state.isChecking = true
    await apiSolver()
    state.isChecking = false
    if(!isRunning.value) return
    if (state.maxUse && state.bytesUsed >= state.maxUse) {
      state.bytesUsed = 0;
      state.logged = 0;
    }
    state.lastLogTime = new Date().getTime() / 1000;
    state.startUse = state.bytesUsed
    state.startTime = new Date().getTime() / 1000;
    state.recordUse = state.bytesUsed
    state.recordTime = new Date().getTime() / 1000;
    for (let i = 0; i < threadNum.value; i++)startThread(i)
    tasks.push(setInterval(frameEvent, 16))
    tasks.push(setInterval(uploadLog, 60000))
    tasks.push(setInterval(apiSolver, 60000))
    secEvent()
    tasks.push(setInterval(secEvent, 1000))
    runBackground.value ? audioDom.value?.play() : ''
  } else {
    tasks.map((i) => console.log(i))
    tasks.map((i) => clearInterval(i))
    tasks = []
    uploadLog()
    audioDom.value?.pause()
    var speed = (state.bytesUsed - state.startUse) / (new Date().getTime() / 1000 - state.startTime)
    setSpeed(speed)
    setUsed()
    if (!props.isVisible) {
      setTitle()
    }
  }
})

async function uploadLog() {
  let now = new Date().getTime() / 1000
  let num = state.bytesUsed - state.logged
  let time = now - state.lastLogTime

  state.logged = state.bytesUsed
  state.lastLogTime = now
  let resp = await fetch(import.meta.env.VITE_API_URL+"log", {
    method: "POST",
    mode: "cors",
    redirect: "follow",
    referrerPolicy: "no-referrer",
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      AccessToken: loginInfo.AccessToken,
      url: runUrl.value,
      threadNum: threadNum.value,
      used: num,
      time: time
    })
  });
  resp = await resp.json()
  if (resp.status == -1) {
    loginInfo.AccessToken = ''
  }
}

watch(props, async (newState, oldState) => {
  if (!newState.isVisible && runBackground.value && isRunning.value) secEvent()
  if (!newState.isVisible && !runBackground.value && isRunning.value) isRunning.value = false
  if (newState.isVisible) setTitle()
})

watch(threadNum, async (newState, oldState) => {
  localStorage.threadNum = newState
  if (isRunning.value && newState > oldState) {
    for (let i = oldState; i < newState; i++)startThread(i)
  }
})

watch(runBackground, async (newState, oldState) => {
  localStorage.runBackground = newState
})

watch(chartShow, async (newState, oldState) => {
  localStorage.chartShow = newState
  if (newState) {
    setTimeout(() => myChart.resize(), 100)
  }
})

watch(runUrl, async (newState, oldState) => {
  localStorage.url = newState
  if (isRunning.value) {
    apiSolver()
  }
})

watch(loginInfo, async (newState, oldState) => {
  localStorage.AccessToken = newState.AccessToken
})

watchEffect(() => {
  localStorage.autoStart = autoStart.value;
})

const copyUrl = () => {
  toClipboard(runUrl.value).then(() => {
    ElMessage.success({
      dangerouslyUseHTMLString: true,
      message: '已复制当前链接',
    })
  })
}

window.addEventListener("paste", function (e) {
  if (!(e.clipboardData && e.clipboardData.items && document.activeElement?.nodeName != 'INPUT'))return;
  for (var i = 0, len = e.clipboardData.items.length; i < len; i++) {
    var itemz = e.clipboardData.items[i];
    if (itemz.type === "text/plain") {
      e.clipboardData.items[i].getAsString(async function (str){
        let clipText = urlParser(str)
        if (clipText) {
          ElMessage.info('读取剪切板链接成功,正在检测链接可用性')
          let ret=await checkUrl(clipText)
          if(ret.status){
            runUrl.value = clipText
            ElMessage.success('读取剪切板链接成功')
          }else{
            ElMessage.error(ret.info)
          }
        } else {
          ElMessage.error('没有检测到剪切板中的链接')
        }
      })
      break
    }
  }
})

var setTitle = (speed: number = 0) => {
  if (props.isVisible) {
    document.title = 'ChuiYan SpeedTest'
  } else {
    if (isRunning.value) {
      document.title = formatter(state.bytesUsed, 0, [0, 0, 0, 0, 0, 0]) + ' ' + formatter(speed, 1, [0, 0, 0, 0, 0, 0])
    } else if (state.maxUse && state.bytesUsed >= state.maxUse) {
      document.title = '已完成'
    } else {
      document.title = '已暂停'
    }
  }
}

var setUsed = () => {
  if (!state.bytesUsed) state.show.allUsed = '-'
  state.show.allUsed = formatter(state.bytesUsed, 0, [0, 0, 1, 2, 2, 2])
}

var setSpeed = (speed: number) => {
  state.show.speed = formatter(speed, 1, [0, 0, 1, 2, 2, 2])
  state.show.speedBit = formatter(speed * 8, 2, [0, 0, 0, 2, 2, 2])
  state.predict.min = formatter(speed * 60, 0, [0, 0, 0, 1, 1, 1])
  state.predict.hour = formatter(speed * 60 * 60, 0, [0, 0, 0, 1, 1, 1])
  state.predict.day = formatter(speed * 60 * 60 * 24, 0, [0, 0, 0, 1, 1, 1])
  state.predict.mon = formatter(speed * 60 * 60 * 24 * 30, 0, [0, 0, 0, 1, 1, 1])
}

var frameEvent = () => {
  if (props.isVisible) setUsed()
  if (state.maxUse && state.bytesUsed >= state.maxUse) {
    isRunning.value = false
  }
}

var secEvent = () => {
  var speed = (state.bytesUsed - state.recordUse) / (new Date().getTime() / 1000 - state.recordTime)
  if (!isNaN(speed)) updateChart(speed)
  else updateChart(0)
  if (speed <= 0 || isNaN(speed)) {
    state.show.speed = '-'
    state.show.speedBit = '-'
  } else if (props.isVisible) {
    setSpeed(speed)
  } else if (runBackground.value) {
    setTitle(speed)
  }
  state.recordUse = state.bytesUsed
  state.recordTime = new Date().getTime() / 1000;
}

function formatter(num: number, desIndex: number, flo: Array<number>) {
  const describeString = [['B', 'KB', 'MB', 'GB', 'TB', 'PB'],
  ['B/s', 'KB/s', 'MB/s', 'GB/s', 'TB/s', 'PB/s'],
  ['Bps', 'Kbps', 'Mbps', 'Gbps', 'Tbps', 'Pbps']
  ]
  const describe = describeString[desIndex]
  var cnum = num;
  var total_index = 0;
  while (cnum >= 1024) {
    if (total_index == describe.length - 1) break;
    cnum = cnum / 1024;
    total_index++;
  }
  return cnum.toFixed(flo[total_index]) + describe[total_index];
}

const speedCtr=()=>{
  if(state.bytesUsed-state.recordUse>state.maxSpeed/8){
    return new Promise((resolve)=>{
      setTimeout(()=>{
        resolve(0)
      },1000-(new Date().getTime()%1000))
    })
  }
}

async function startThread(index: number) {
  try {
    if(solvedRunUrl==""){
      isRunning.value=false
      return
    }
    let _url=solvedRunUrl
    const response = await fetch(_url, { cache: "no-store", mode: 'cors', referrerPolicy: 'no-referrer' })
    if (!response.body) throw "Nobody"
    let contentLength = response.headers.get('content-length')
    let realLength = Infinity
    if (contentLength) realLength = parseInt(contentLength)
    const reader = response.body.getReader();
    let decodeLength = 0
    while (true) {
      if(state.maxSpeed)await speedCtr()
      const { value } = await reader.read();
      let chunkLength = value?.length
      if (!chunkLength || solvedRunUrl != _url) {
        startThread(index);
        break;
      }
      let usefulChunkLength = chunkLength
      if (decodeLength >= realLength) {
        usefulChunkLength = 0
      } else if (decodeLength + chunkLength > realLength) {
        usefulChunkLength = realLength - decodeLength
      }
      state.bytesUsed += usefulChunkLength
      if (index >= threadNum.value || !isRunning.value) break
      decodeLength += chunkLength
    }
    reader.cancel()
  } catch (err) {
    console.log(err)
    if (isRunning.value) startThread(index);
  }
}

const EditTableVisible = ref(false)
const addTableVisible = ref(false)
const EditMaxVisible = ref(false)
const EditSpeedVisible = ref(false)
const addForm = ref({
  label: '',
  value: '',
  checking: false
})

const urlParser = (ipt: string) => {
  var a = ipt.match(/https?:\/\/([\w-]+\.)+[\w-]+(:[0-9]+)?(\/\S*)?/);
  return a ? a[0] : '';
}

const addNode = async () => {
  addForm.value.value = urlParser(addForm.value.value)
  addForm.value.checking = true
  const urlStatus = await checkUrl(addForm.value.value)
  if (!urlStatus.status) {
    addForm.value.checking = false
    ElMessage.error({
      dangerouslyUseHTMLString: true,
      message: urlStatus.info,
    })
    return
  }
  addForm.value.checking = false
  customNodes.push({
    label: addForm.value.label,
    value: addForm.value.value,
  })
  addForm.value.label = ''
  addForm.value.value = ''
  addTableVisible.value = false
}

const maxUseInput: Ref<{
  num: number | null;
  type: "GB" | "MB" | "TB";
}> = ref({
  num: null,
  type: 'GB',
})

const editMaxUse = () => {
  var map = {
    "MB": 1024 * 1024,
    "GB": 1024 * 1024 * 1024,
    "TB": 1024 * 1024 * 1024 * 1024
  }
  var tmp = 0
  if (maxUseInput.value.num) {
    tmp = Math.floor(maxUseInput.value.num * map[maxUseInput.value.type])
  }
  state.maxUse = tmp
  localStorage.maxUse = tmp
  maxUseInput.value.num = null
  EditMaxVisible.value = false
}

const maxSpeedInput: Ref<{
  num: number | null;
  type: "Gbps" | "Mbps" | "Kbps";
}> = ref({
  num: null,
  type: 'Mbps',
})

const editSpeedUse = () => {
  var map = {
    "Kbps": 1024 ,
    "Mbps": 1024 * 1024,
    "Gbps": 1024 * 1024 * 1024
  }
  var tmp = 0
  if (maxSpeedInput.value.num) {
    tmp = Math.floor(maxSpeedInput.value.num * map[maxSpeedInput.value.type])
  }
  state.maxSpeed = tmp
  localStorage.maxSpeed = tmp
  maxSpeedInput.value.num = null
  EditSpeedVisible.value = false
}

var isMobile = /Mobi|Android|iPhone|Macintosh/i.test(navigator.userAgent)
var isMiuiBrowser = /MiuiBrowser/i.test(navigator.userAgent)
var isIOS = /iPhone|Macintosh/i.test(navigator.userAgent)

const audioDom: Ref<any> = ref(null);
const chartContainer = ref(null);

let myChart: EChartsType;
let updateChart = (n:number) => {};
let clearChart=()=>{};

onMounted(() => {
  myChart = echarts.init(chartContainer.value);
  const chartOption = {
    tooltip: {
      trigger: 'axis',
      formatter: function (params: any) {
        let speed = formatter(params[0].data[1], 1, [0, 0, 1, 2, 2, 2]);
        return `${new Date(params[0].data[0] * 1000).toLocaleString()}<br />
              ${speed}`;
      },
    },
    toolbox: {
      feature: {
        saveAsImage: {}
      }
    },
    title: {
      left: 'left',
      text: '速度图表'
    },
    xAxis: {
      type: 'time',
      boundaryGap: false,
      axisLabel: {
        show: false
      },
      axisTick:{
        show:false
      }
    },
    yAxis: {
      type: 'value',
      axisLabel: {
        formatter: (val: number) => {
          let a = formatter(val, 1, [0, 0, 0, 0, 0, 0]);
          return a == '-' ? 0 : a
        }
      }
    },
    series: [
      {
        name: '速度',
        type: 'line',
        smooth: false,
        symbol: 'none',
        areaStyle: {},
        data: [[new Date().getTime() / 1000,0]]
      }
    ],
    grid: {
      x: 0,
      y: 40,
      x2: 8,
      y2: 10,
      containLabel: true
    },
  }

  myChart.setOption(chartOption);
  let showArray:Array<any>=[]
  let speedTemp:Array<number>=[]
  let stepLength=1
  clearChart=()=>{
    speedTemp=[]
    showArray.push([new Date().getTime() / 1000,0])
  }
  updateChart = (speed:number) => {
    let refresh=false
    speedTemp.push(speed)
    while(speedTemp.length>=stepLength){
      refresh=true
      var tmp = speedTemp.splice(0, stepLength);
      let avg;
      if(tmp.includes(0))avg=0
      else avg = tmp.reduce((a, b) => a + b,0)/stepLength;
      showArray.push([new Date().getTime() / 1000,avg])
    }
    while(showArray.length>=200){
      refresh=true
      const result = [];
      const lengthToProcess = showArray.length % 2 === 0 ? showArray.length : showArray.length - 1;
      for (let i = 0; i < lengthToProcess; i += 2) {
        result.push([showArray[i][0],(showArray[i][1] + showArray[i + 1][1]) / 2]);
      }
      showArray=result
      stepLength*=2
    }
    chartOption.series[0].data = showArray
    if(chartShow.value && refresh)myChart.setOption(chartOption);
  }
  window.addEventListener('resize', () => { myChart.resize() });
});

onUnmounted(() => {
  if (myChart) {
    myChart.dispose();
  }
});
</script>

<style scoped>
.main-container {
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
}

/* iOS 卡片样式 */
.ios-card {
  backdrop-filter: blur(20px);
  background: rgba(255, 255, 255, 0.7);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  padding: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
  transition: all 0.3s ease;
}

.ios-card:hover {
  background: rgba(255, 255, 255, 0.8);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.12);
}

@media (prefers-color-scheme: dark) {
  .ios-card {
    background: rgba(26, 26, 46, 0.6);
    border-color: rgba(255, 255, 255, 0.1);
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  }

  .ios-card:hover {
    background: rgba(26, 26, 46, 0.7);
    box-shadow: 0 12px 40px rgba(0, 0, 0, 0.4);
  }
}

/* 卡片头部 */
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.card-title {
  font-size: 16px;
  font-weight: 600;
  color: #333;
}

@media (prefers-color-scheme: dark) {
  .card-title {
    color: #e2e8f0;
  }
}

.card-value {
  font-size: 18px;
  font-weight: 700;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.card-actions {
  display: flex;
  gap: 8px;
}

/* 图标按钮 */
.icon-btn {
  width: 32px;
  height: 32px;
  border-radius: 8px;
  border: none;
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  padding: 0;
}

.icon-btn:hover {
  background: rgba(102, 126, 234, 0.2);
  transform: scale(1.05);
}

.icon-btn svg {
  width: 18px;
  height: 18px;
}

@media (prefers-color-scheme: dark) {
  .icon-btn {
    background: rgba(160, 174, 192, 0.1);
    color: #a0aec0;
  }

  .icon-btn:hover {
    background: rgba(160, 174, 192, 0.2);
  }
}

.icon-btn-small {
  width: 24px;
  height: 24px;
  border-radius: 6px;
  border: none;
  background: transparent;
  color: #667eea;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  padding: 0;
}

.icon-btn-small:hover {
  background: rgba(102, 126, 234, 0.1);
}

.icon-btn-small svg {
  width: 16px;
  height: 16px;
}

@media (prefers-color-scheme: dark) {
  .icon-btn-small {
    color: #a0aec0;
  }

  .icon-btn-small:hover {
    background: rgba(160, 174, 192, 0.1);
  }
}

/* iOS 选择框 */
.ios-select-wrapper {
  width: 100%;
}

.ios-select {
  width: 100%;
}

:deep(.ios-select .el-input__wrapper) {
  background: rgba(0, 0, 0, 0.05);
  border-radius: 12px;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

:deep(.ios-select .el-input__wrapper:hover) {
  background: rgba(0, 0, 0, 0.08);
}

@media (prefers-color-scheme: dark) {
  :deep(.ios-select .el-input__wrapper) {
    background: rgba(255, 255, 255, 0.05);
    border-color: rgba(255, 255, 255, 0.1);
  }

  :deep(.ios-select .el-input__wrapper:hover) {
    background: rgba(255, 255, 255, 0.08);
  }
}

/* iOS 滑块 */
.ios-slider-wrapper {
  width: 100%;
}

:deep(.ios-slider .el-slider__bar) {
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
}

:deep(.ios-slider .el-slider__button) {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: white;
  border: 3px solid #667eea;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.3);
}

:deep(.ios-slider .el-slider__track) {
  background: rgba(102, 126, 234, 0.2);
}

@media (prefers-color-scheme: dark) {
  :deep(.ios-slider .el-slider__button) {
    background: #1a1a2e;
    border-color: #a0aec0;
  }

  :deep(.ios-slider .el-slider__track) {
    background: rgba(160, 174, 192, 0.2);
  }
}

/* iOS 开关 */
.ios-switch {
  --el-switch-on-color: #667eea;
  --el-switch-off-color: #ccc;
}

:deep(.ios-switch .el-switch__core) {
  border-radius: 12px;
  width: 50px;
  height: 30px;
}

:deep(.ios-switch .el-switch__inner) {
  font-size: 12px;
}

@media (prefers-color-scheme: dark) {
  .ios-switch {
    --el-switch-off-color: #666;
  }
}

/* 设置网格 */
.settings-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.setting-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  background: rgba(0, 0, 0, 0.02);
  border-radius: 12px;
}

@media (prefers-color-scheme: dark) {
  .setting-item {
    background: rgba(255, 255, 255, 0.02);
  }
}

.setting-label {
  font-size: 14px;
  color: #333;
  font-weight: 500;
}

@media (prefers-color-scheme: dark) {
  .setting-label {
    color: #e2e8f0;
  }
}

@media (max-width: 640px) {
  .settings-grid {
    grid-template-columns: 1fr;
  }
}

/* 数据网格 */
.data-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

@media (max-width: 768px) {
  .data-grid {
    grid-template-columns: 1fr;
  }
}

.data-item {
  padding: 16px;
  background: rgba(0, 0, 0, 0.02);
  border-radius: 12px;
  border: 1px solid rgba(0, 0, 0, 0.05);
}

@media (prefers-color-scheme: dark) {
  .data-item {
    background: rgba(255, 255, 255, 0.02);
    border-color: rgba(255, 255, 255, 0.05);
  }
}

.data-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  gap: 8px;
}

.data-label {
  font-size: 13px;
  color: #666;
  font-weight: 500;
}

@media (prefers-color-scheme: dark) {
  .data-label {
    color: #a0aec0;
  }
}

.data-limit {
  font-size: 12px;
  color: #999;
}

@media (prefers-color-scheme: dark) {
  .data-limit {
    color: #718096;
  }
}

.data-value {
  font-size: 28px;
  font-weight: 700;
  color: #333;
  word-break: break-word;
}

@media (prefers-color-scheme: dark) {
  .data-value {
    color: #e2e8f0;
  }
}

.speed-value {
  color: #667eea;
}

/* 预测弹窗 */
.predict-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.predict-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.predict-label {
  font-size: 12px;
  color: #666;
}

@media (prefers-color-scheme: dark) {
  .predict-label {
    color: #a0aec0;
  }
}

.predict-value {
  font-size: 14px;
  font-weight: 600;
  color: #333;
}

@media (prefers-color-scheme: dark) {
  .predict-value {
    color: #e2e8f0;
  }
}

/* 控制卡片 */
.control-card {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.control-buttons {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 24px;
  flex-wrap: wrap;
}

/* 播放按钮 */
.play-btn {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  border: none;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32px;
  box-shadow: 0 8px 24px rgba(102, 126, 234, 0.4);
  transition: all 0.3s ease;
}

.play-btn:hover:not(:disabled) {
  transform: scale(1.1);
  box-shadow: 0 12px 32px rgba(102, 126, 234, 0.5);
}

.play-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.play-btn.pause {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  box-shadow: 0 8px 24px rgba(245, 87, 108, 0.4);
}

.play-btn.pause:hover {
  box-shadow: 0 12px 32px rgba(245, 87, 108, 0.5);
}

.play-btn.loading {
  background: linear-gradient(135deg, #ffa751 0%, #ffe259 100%);
  box-shadow: 0 8px 24px rgba(255, 167, 81, 0.4);
}

.play-btn svg {
  width: 40px;
  height: 40px;
}

.spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* 控制图标按钮 */
.control-icons {
  display: flex;
  gap: 12px;
}

.control-icon-btn {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  border: none;
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  padding: 0;
}

.control-icon-btn:hover {
  background: rgba(102, 126, 234, 0.2);
  transform: scale(1.05);
}

.control-icon-btn svg {
  width: 24px;
  height: 24px;
}

@media (prefers-color-scheme: dark) {
  .control-icon-btn {
    background: rgba(160, 174, 192, 0.1);
    color: #a0aec0;
  }

  .control-icon-btn:hover {
    background: rgba(160, 174, 192, 0.2);
  }
}

/* 图表容器 */
.chart-container {
  width: 100%;
  height: 400px;
  border-radius: 12px;
  overflow: hidden;
  background: rgba(0, 0, 0, 0.02);
}

@media (prefers-color-scheme: dark) {
  .chart-container {
    background: rgba(255, 255, 255, 0.02);
  }
}

/* 对话框样式 */
:deep(.el-dialog) {
  border-radius: 20px !important;
}

:deep(.el-dialog__header) {
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

@media (prefers-color-scheme: dark) {
  :deep(.el-dialog__header) {
    border-bottom-color: rgba(255, 255, 255, 0.05);
  }
}

:deep(.el-dialog__body) {
  padding: 24px;
}

:deep(.el-input__wrapper) {
  border-radius: 12px;
}

:deep(.el-button) {
  border-radius: 12px;
}

:deep(.el-button--primary) {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
}

:deep(.el-button--primary:hover) {
  opacity: 0.9;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .ios-card {
    padding: 16px;
    border-radius: 16px;
  }

  .control-buttons {
    gap: 16px;
  }

  .play-btn {
    width: 72px;
    height: 72px;
    font-size: 28px;
  }

  .play-btn svg {
    width: 36px;
    height: 36px;
  }

  .control-icon-btn {
    width: 44px;
    height: 44px;
  }

  .data-value {
    font-size: 24px;
  }
}

@media (max-width: 480px) {
  .main-container {
    gap: 12px;
  }

  .ios-card {
    padding: 12px;
    border-radius: 12px;
  }

  .card-header {
    margin-bottom: 12px;
  }

  .card-title {
    font-size: 14px;
  }

  .data-grid {
    gap: 12px;
  }

  .data-item {
    padding: 12px;
  }

  .data-value {
    font-size: 20px;
  }

  .play-btn {
    width: 64px;
    height: 64px;
    font-size: 24px;
  }

  .play-btn svg {
    width: 32px;
    height: 32px;
  }

  .control-icon-btn {
    width: 40px;
    height: 40px;
  }

  .control-icon-btn svg {
    width: 20px;
    height: 20px;
  }
}
</style>
