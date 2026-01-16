<template>
  <div class="main-container">
    <div class="ios-card glass">
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
        <el-select v-model="runUrl" class="ios-select-custom">
          <el-option-group v-for="group in nodes" :key="group.label" :label="group.label">
            <el-option v-for="item in group.options" :key="item.value" :label="item.label" :value="item.value" />
          </el-option-group>
        </el-select>
      </div>
    </div>

    <div class="ios-card glass">
      <div class="card-header">
        <span class="card-title">线程数</span>
        <span class="card-value">{{ threadNum }}</span>
      </div>
      <div class="ios-slider-wrapper">
        <el-slider :show-tooltip="false" :min="1" :max="64" v-model="threadNum" class="ios-slider-custom" />
      </div>
      <div class="mini-settings">
        <div class="mini-setting-item">
          <span class="mini-label">后台运行</span>
          <el-switch v-model="runBackground" class="ios-switch-custom" />
        </div>
        <div class="mini-setting-item">
          <span class="mini-label">自动运行</span>
          <el-switch v-model="autoStart" class="ios-switch-custom" />
        </div>
      </div>
    </div>

    <div class="ios-card glass">
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

        <div class="data-item highlight">
          <div class="data-header">
            <span class="data-label">{{ isRunning ? '实时速度' : '平均速度' }}</span>
            <el-popover placement="top-start" title="用量预测" :width="180" trigger="click" popper-class="ios-popper">
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

    <div class="ios-card glass control-card">
      <div class="control-row">
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

        <button class="control-icon-btn" @click="chartShow = !chartShow" title="图表">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polyline points="12 3 20 7.5 20 16.5 12 21 4 16.5 4 7.5 12 3"></polyline>
            <polyline points="12 12 20 7.5"></polyline>
            <polyline points="12 12 12 21"></polyline>
            <polyline points="12 12 4 7.5"></polyline>
          </svg>
        </button>
        <button class="control-icon-btn" @click="isFullScreen = true" title="全屏">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3"></path>
          </svg>
        </button>
      </div>
      <div v-show="chartShow" ref="chartContainer" class="chart-container"></div>
    </div>

    <el-dialog v-model="EditTableVisible" title="自定义地址" class="ios-dialog-custom" append-to-body>
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
      <el-button class="mt-4 ios-btn-full" type="primary" @click="addTableVisible = true;">添加地址</el-button>
    </el-dialog>

    <el-dialog v-model="addTableVisible" title="添加链接" class="ios-dialog-custom" append-to-body>
      <el-form :model="addForm" label-position="top">
        <el-form-item label="名称">
          <el-input v-model="addForm.label" autocomplete="off" placeholder="节点名称" />
        </el-form-item>
        <el-form-item label="URL">
          <el-input v-model="addForm.value" autocomplete="off" placeholder="https://...">
            <template #suffix>
              <el-icon v-if="urlParser(addForm.value)" color="#34c759">
                <Check />
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
      </el-form>
      <div class="alert-container">
        <el-alert title="注意" type="warning" :closable="false" show-icon>
          在浏览器工作的程序受到浏览器安全策略的限制。
        </el-alert>
        <div class="alert-spacer"></div>
        <el-alert title="免责声明" type="error" :closable="false" show-icon>
          请勿用于非法用途，使用本功能造成的一切后果由用户承担。
        </el-alert>
      </div>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="addTableVisible = false">取消</el-button>
          <el-button type="primary" :disabled="!urlParser(addForm.value) || !addForm.label || addForm.checking"
            @click="addNode()">确认
          </el-button>
        </div>
      </template>
    </el-dialog>

    <el-dialog v-model="EditMaxVisible" title="设置上限自动停止" class="ios-dialog-custom mini-dialog" append-to-body>
      <div class="input-with-select">
        <el-input type="number" min="1" v-model="maxUseInput.num" placeholder="留空则无上限" />
        <el-select v-model="maxUseInput.type" style="width: 100px">
          <el-option label="MB" value="MB" />
          <el-option label="GB" value="GB" />
          <el-option label="TB" value="TB" />
        </el-select>
      </div>
      <template #footer>
        <el-button type="primary" @click="editMaxUse()" class="ios-btn-full">确定</el-button>
      </template>
    </el-dialog>

    <el-dialog v-model="EditSpeedVisible" title="设置带宽上限" class="ios-dialog-custom mini-dialog" append-to-body>
      <div class="input-with-select">
        <el-input type="number" min="1" v-model="maxSpeedInput.num" placeholder="留空则无上限" />
        <el-select v-model="maxSpeedInput.type" style="width: 100px">
          <el-option label="Mbps" value="Mbps" />
          <el-option label="Gbps" value="Gbps" />
        </el-select>
      </div>
      <template #footer>
        <el-button type="primary" @click="editSpeedUse()" class="ios-btn-full">确定</el-button>
      </template>
    </el-dialog>

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
import { ElMessage } from 'element-plus'
import nodesJson from "../assets/nodes.json"
import { ref, watch, watchEffect, type Ref, reactive, onMounted, onUnmounted } from 'vue'
import { toClipboard } from '@soerenmartius/vue3-clipboard'
import FullScreenUI from './FullScreen.vue'
import * as echarts from 'echarts';
import { Check } from '@element-plus/icons-vue'

const props = defineProps({
  isVisible: Boolean
})

const customNodes = reactive(localStorage.customNodes ? JSON.parse(localStorage.customNodes) : [])
const OnlineNodes: any[] = []
for(let groupName in nodesJson) {
  const group=nodesJson[groupName as keyof typeof nodesJson]
  const temp: any ={"label":groupName,options:[]}
  for(let node in group) {
    temp.options.push({"value":group[node as keyof typeof group],"label":node})
  }
  OnlineNodes.push(temp)
}
const nodes: Ref<any[]> = ref(OnlineNodes)
if (customNodes.length) {
  nodes.value = [{ "label": "自定义", "options": customNodes}].concat(OnlineNodes)
}
watch(customNodes, async (newState) => {
  if (customNodes.length) {
    nodes.value = [{ "label": "自定义", "options": customNodes }].concat(OnlineNodes)
  } else nodes.value = OnlineNodes
  localStorage.customNodes = JSON.stringify(newState)
}, { deep: true })

const state = reactive({
  show: { allUsed: '-', speed: '-', speedBit: '-' },
  predict: { min: '-', hour: '-', day: '-', mon: '-' },
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
const chartShow = ref(localStorage.chartShow ? localStorage.chartShow === 'true' : false)
const threadNum = ref(localStorage.threadNum ? Number(localStorage.threadNum) : 8)
const runBackground = ref(localStorage.runBackground ? localStorage.runBackground === 'true' : false)
const autoStart = ref(localStorage.autoStart ? localStorage.autoStart === 'true' : false)
const runUrl = ref(localStorage.url ? localStorage.url : nodes.value[0].options[0].value)

var tasks: Array<number> = []

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
    ElMessage.error({ dangerouslyUseHTMLString: true, message: urlStatus.info })
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
    if (block_list.some((i)=>structUrl.host.endsWith(i))) throw '你不对劲！'
    const controller = new AbortController();
    setTimeout(() => controller.abort(), 5000);
    const response = await fetch(url, { cache: "no-store", mode: 'cors', referrerPolicy: 'no-referrer' ,signal: controller.signal})
    if (response.status == 404) throw "资源响应异常 404"
    if (!response.body) throw "资源响应异常 Nobody"
    const reader = response.body.getReader();
    const { value } = await reader.read();
    if (!value || value.length <= 0) throw "资源响应异常 Nobody";
    reader.cancel()
  } catch (err) {
    status = false
    info = err instanceof Error ? err.message : String(err)
  }
  return { status, info }
}

let solvedRunUrl = ''
async function apiSolver(){
  if(!runUrl.value.startsWith("NetworkPanelApi://")){
    solvedRunUrl = runUrl.value
    return
  }
  let host=runUrl.value.split("NetworkPanelApi://")[1]
  let resp:any = await fetch(import.meta.env.VITE_API_URL+"url.ajax?"+new URLSearchParams({host:host,cache:window.location.host}), {
      mode: "cors", redirect: "follow", referrerPolicy: "no-referrer"
    });
  resp = await resp.json()
  if(resp['status']!=0){ isRunning.value=false; return }
  solvedRunUrl = resp['url']
}

watch(isRunning, async (newState) => {
  clearChart()
  if (newState) {
    state.isChecking = true
    await apiSolver()
    state.isChecking = false
    if(!isRunning.value) return
    if (state.maxUse && state.bytesUsed >= state.maxUse) {
      state.bytesUsed = 0; state.logged = 0;
    }
    state.lastLogTime = new Date().getTime() / 1000;
    state.startUse = state.bytesUsed
    state.startTime = new Date().getTime() / 1000;
    state.recordUse = state.bytesUsed
    state.recordTime = new Date().getTime() / 1000;
    for (let i = 0; i < threadNum.value; i++)startThread(i)
    tasks.push(setInterval(frameEvent, 16))
    tasks.push(setInterval(apiSolver, 60000))
    secEvent()
    tasks.push(setInterval(secEvent, 1000))
    runBackground.value ? audioDom.value?.play() : ''
  } else {
    tasks.map((i) => clearInterval(i))
    tasks = []
    audioDom.value?.pause()
    var speed = (state.bytesUsed - state.startUse) / (new Date().getTime() / 1000 - state.startTime)
    setSpeed(speed)
    setUsed()
    if (!props.isVisible) setTitle()
  }
})

watch(props, async (newState) => {
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

watch(runBackground, async (newState) => { localStorage.runBackground = newState })
watch(chartShow, async (newState) => {
  localStorage.chartShow = newState
  if (newState) setTimeout(() => myChart.resize(), 100)
})
watch(runUrl, async () => { localStorage.url = runUrl.value; if (isRunning.value) apiSolver() })
watchEffect(() => { localStorage.autoStart = autoStart.value; })

const copyUrl = () => {
  toClipboard(runUrl.value).then(() => ElMessage.success('已复制当前链接'))
}

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
  if (state.maxUse && state.bytesUsed >= state.maxUse) isRunning.value = false
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
      setTimeout(()=>{ resolve(0) },1000-(new Date().getTime()%1000))
    })
  }
}

async function startThread(index: number) {
  try {
    if(solvedRunUrl==""){ isRunning.value=false; return }
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
        startThread(index); break;
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
    if (isRunning.value) startThread(index);
  }
}

const EditTableVisible = ref(false)
const addTableVisible = ref(false)
const EditMaxVisible = ref(false)
const EditSpeedVisible = ref(false)
const addForm = ref({ label: '', value: '', checking: false })

const urlParser = (ipt: string) => {
  var a = ipt.match(/https?:\/\/([\w-]+\.)+[\w-]+(:[0-9]+)?(\/\S*)?/);
  return a ? a[0] : '';
}

const addNode = async () => {
  addForm.value.value = urlParser(addForm.value.value)
  addForm.value.checking = true
  const urlStatus = await checkUrl(addForm.value.value)
  addForm.value.checking = false
  if (!urlStatus.status) {
    ElMessage.error({ dangerouslyUseHTMLString: true, message: urlStatus.info })
    return
  }
  customNodes.push({ label: addForm.value.label, value: addForm.value.value })
  addForm.value.label = ''; addForm.value.value = ''; addTableVisible.value = false
}

const maxUseInput: Ref<any> = ref({ num: null, type: 'GB' })
const editMaxUse = () => {
  var map: any = { "MB": 1024 * 1024, "GB": 1024 * 1024 * 1024, "TB": 1024 * 1024 * 1024 * 1024 }
  var tmp = 0
  if (maxUseInput.value.num) tmp = Math.floor(maxUseInput.value.num * map[maxUseInput.value.type])
  state.maxUse = tmp; localStorage.maxUse = tmp; maxUseInput.value.num = null; EditMaxVisible.value = false
}

const maxSpeedInput: Ref<any> = ref({ num: null, type: 'Mbps' })
const editSpeedUse = () => {
  var map: any = { "Kbps": 1024 , "Mbps": 1024 * 1024, "Gbps": 1024 * 1024 * 1024 }
  var tmp = 0
  if (maxSpeedInput.value.num) tmp = Math.floor(maxSpeedInput.value.num * map[maxSpeedInput.value.type])
  state.maxSpeed = tmp; localStorage.maxSpeed = tmp; maxSpeedInput.value.num = null; EditSpeedVisible.value = false
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
  const isDark = window.matchMedia("(prefers-color-scheme: dark)").matches;
  const chartOption = {
    backgroundColor: 'transparent',
    toolbox: {
      show: true,
      feature: {
        saveAsImage: {
          show: true,
          title: '下载',
          iconStyle: { 
            borderColor: isDark ? '#ffffff' : '#1c1c1e',
            borderWidth: 1.5
          }
        }
      },
      right: 10,
      top: 0
    },
    tooltip: {
      trigger: 'axis',
      backgroundColor: isDark ? 'rgba(28, 28, 30, 0.9)' : 'rgba(255, 255, 255, 0.9)',
      borderColor: 'transparent',
      textStyle: { color: isDark ? '#ffffff' : '#1c1c1e' },
      formatter: function (params: any) {
        let speed = formatter(params[0].data[1], 1, [0, 0, 1, 2, 2, 2]);
        return `${new Date(params[0].data[0] * 1000).toLocaleTimeString()}<br />${speed}`;
      },
    },
    xAxis: {
      type: 'time',
      boundaryGap: false,
      axisLabel: { show: false },
      axisTick:{ show:false },
      splitLine: { show: false }
    },
    yAxis: {
      type: 'value',
      axisLabel: {
        color: isDark ? '#ffffff' : '#3a3a3c',
        formatter: (val: number) => {
          let a = formatter(val, 1, [0, 0, 0, 0, 0, 0]);
          return a == '-' ? 0 : a
        }
      },
      splitLine: { lineStyle: { color: isDark ? 'rgba(255,255,255,0.1)' : 'rgba(0,0,0,0.05)' } }
    },
    series: [
      {
        name: '速度',
        type: 'line',
        smooth: true,
        symbol: 'none',
        lineStyle: { color: '#007aff', width: 3 },
        areaStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(0, 122, 255, 0.3)' },
            { offset: 1, color: 'rgba(0, 122, 255, 0)' }
          ])
        },
        data: [[new Date().getTime() / 1000,0]]
      }
    ],
    grid: { x: 10, y: 30, x2: 10, y2: 10, containLabel: true },
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
      let avg = tmp.includes(0) ? 0 : tmp.reduce((a, b) => a + b,0)/stepLength;
      showArray.push([new Date().getTime() / 1000,avg])
    }
    while(showArray.length>=200){
      refresh=true
      const result = [];
      const lengthToProcess = showArray.length % 2 === 0 ? showArray.length : showArray.length - 1;
      for (let i = 0; i < lengthToProcess; i += 2) {
        result.push([showArray[i][0],(showArray[i][1] + showArray[i + 1][1]) / 2]);
      }
      showArray=result; stepLength*=2
    }
    chartOption.series[0].data = showArray
    if(chartShow.value && refresh)myChart.setOption(chartOption);
  }
  window.addEventListener('resize', () => { myChart.resize() });
});

onUnmounted(() => { if (myChart) myChart.dispose(); });
</script>

<style scoped>
.main-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
}

.ios-card {
  border-radius: 24px;
  padding: 24px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.glass {
  background: var(--glass-bg);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border: 1px solid var(--glass-stroke);
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.04);
}

@media (prefers-color-scheme: dark) {
  .glass {
    box-shadow: 0 4px 24px rgba(0, 0, 0, 0.2);
  }
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.card-title {
  font-size: 17px;
  font-weight: 600;
  color: var(--text-main);
  letter-spacing: -0.4px;
}

.card-value {
  font-size: 17px;
  font-weight: 600;
  color: var(--ios-blue);
}

.card-actions {
  display: flex;
  gap: 12px;
}

.icon-btn {
  width: 36px;
  height: 36px;
  border-radius: 10px;
  border: none;
  background: rgba(0, 122, 255, 0.1);
  color: var(--ios-blue);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.icon-btn:active {
  transform: scale(0.9);
  background: rgba(0, 122, 255, 0.2);
}

.icon-btn svg {
  width: 20px;
  height: 20px;
}

.ios-select-custom {
  width: 100%;
}

.mini-settings {
  display: flex;
  gap: 20px;
  margin-top: 20px;
  padding-top: 16px;
  border-top: 1px solid rgba(0, 0, 0, 0.05);
}

@media (prefers-color-scheme: dark) {
  .mini-settings {
    border-top-color: rgba(255, 255, 255, 0.05);
  }
}

.mini-setting-item {
  display: flex;
  align-items: center;
  gap: 10px;
}

.mini-label {
  font-size: 14px;
  color: var(--text-secondary);
  font-weight: 600;
}

.ios-switch-custom {
  --el-switch-on-color: #34c759 !important;
}

:deep(.el-switch.is-checked .el-switch__core) {
  background-color: #34c759 !important;
  border-color: #34c759 !important;
}

:deep(.el-switch__core) {
  border: none !important;
  background: rgba(120, 120, 128, 0.16) !important;
}

.data-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

.data-item {
  padding: 20px;
  background: rgba(0, 0, 0, 0.03);
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

@media (prefers-color-scheme: dark) {
  .data-item {
    background: rgba(255, 255, 255, 0.04);
  }
}

.data-item.highlight {
  background: rgba(0, 122, 255, 0.08);
  border: 1px solid rgba(0, 122, 255, 0.1);
}

.data-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.data-label {
  font-size: 13px;
  color: var(--text-secondary);
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.icon-btn-small {
  background: none;
  border: none;
  padding: 4px;
  color: var(--text-secondary);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  transition: background 0.2s;
}

.icon-btn-small:hover {
  background: rgba(0, 0, 0, 0.05);
}

.icon-btn-small svg {
  width: 14px;
  height: 14px;
}

.data-value {
  font-size: 24px;
  font-weight: 700;
  color: var(--text-main);
  letter-spacing: -0.5px;
}

.speed-value {
  color: var(--ios-blue);
}

.control-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 24px;
}

.play-btn {
  width: 72px;
  height: 72px;
  border-radius: 50%;
  border: none;
  background: var(--ios-blue);
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 20px rgba(0, 122, 255, 0.3);
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.play-btn:active {
  transform: scale(0.85);
}

.play-btn.pause {
  background: #ff3b30;
  box-shadow: 0 8px 20px rgba(255, 59, 48, 0.3);
}

.play-btn svg {
  width: 32px;
  height: 32px;
}

.control-icon-btn {
  width: 44px;
  height: 44px;
  border-radius: 14px;
  border: none;
  background: rgba(0, 0, 0, 0.05);
  color: var(--text-main);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
  padding: 0;
  flex-shrink: 0;
  -webkit-appearance: none;
}

.control-icon-btn svg {
  width: 24px;
  height: 24px;
  display: block;
}

@media (prefers-color-scheme: dark) {
  .control-icon-btn {
    background: rgba(255, 255, 255, 0.1);
  }
}

.control-icon-btn:active {
  transform: scale(0.9);
}

.chart-container {
  width: 100%;
  height: 300px;
  margin-top: 20px;
  border-radius: 16px;
  background: transparent;
}

.alert-container {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
}

.alert-spacer {
  height: 12px;
}

:deep(.el-alert) {
  border-radius: 14px !important;
  padding: 12px 16px !important;
}

.ios-btn-full {
  width: 100%;
  height: 44px;
  border-radius: 12px !important;
  font-weight: 600 !important;
}

@media (max-width: 600px) {
  .data-grid {
    grid-template-columns: 1fr;
    gap: 12px;
  }
  .data-item {
    padding: 16px;
    gap: 8px;
  }
  .data-value {
    font-size: 20px;
  }
  .ios-card {
    padding: 16px;
    border-radius: 20px;
  }
  .mini-settings {
    gap: 12px;
  }
}

@media (max-width: 360px) {
  .mini-settings {
    flex-direction: column;
    align-items: flex-start;
  }
  .control-row {
    gap: 16px;
  }
}
</style>

<style scoped>
.input-with-select {
  display: flex;
  gap: 10px;
  align-items: center;
}
.input-with-select .el-input {
  flex: 1;
}
</style>
