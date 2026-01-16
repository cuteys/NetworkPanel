<template>
  <div class="ios-card glass">
    <div class="ios-section">
      <div class="ios-row">
        <span class="ios-label">测速地址</span>
        <div class="ios-actions">
          <el-button type="primary" :icon="CopyDocument" link @click="copyUrl" />
          <el-button type="primary" :icon="Edit" link @click="EditTableVisible = true" />
        </div>
      </div>
      <el-select class="ios-select" v-model="runUrl" placeholder="选择节点">
        <el-option-group v-for="group in nodes" :key="group.label" :label="group.label">
          <el-option v-for="item in group.options" :key="item.value" :label="item.label" :value="item.value" />
        </el-option-group>
      </el-select>
    </div>

    <div class="ios-section">
      <div class="ios-row">
        <span class="ios-label">线程数</span>
        <span class="ios-value">{{ threadNum }}</span>
      </div>
      <el-slider class="ios-slider" :show-tooltip="false" :min="1" :max="64" v-model="threadNum" />
    </div>

    <div class="ios-section ios-switches">
      <div class="ios-switch-item">
        <span>保持后台运行</span>
        <el-switch v-model="runBackground" />
      </div>
      <div class="ios-switch-item">
        <span>自动运行</span>
        <el-switch v-model="autoStart" />
      </div>
    </div>

    <div class="ios-stats-grid">
      <div class="ios-stat-box">
        <div class="stat-header">
          <span class="stat-label">总流量</span>
          <el-button type="primary" :icon="Edit" link @click="EditMaxVisible = true" />
        </div>
        <div class="stat-value">{{ state.show.allUsed }}</div>
        <div class="stat-sub" v-if="state.maxUse">/ {{ formatter(state.maxUse, 0, [0, 0, 0, 0, 0, 0]) }}</div>
      </div>
      <div class="ios-stat-box highlight">
        <div class="stat-header">
          <span class="stat-label">{{ isRunning ? '实时速度' : '平均速度' }}</span>
          <el-popover placement="top" title="用量预测" :width="150" trigger="click">
            <template #reference>
              <el-button type="primary" :icon="Calendar" link />
            </template>
            <div class="predict-list">
              <div>分: {{ state.predict.min }}</div>
              <div>时: {{ state.predict.hour }}</div>
              <div>天: {{ state.predict.day }}</div>
            </div>
          </el-popover>
        </div>
        <div class="stat-value">{{ state.show.speed }}</div>
      </div>
      <div class="ios-stat-box">
        <div class="stat-header">
          <span class="stat-label">带宽</span>
          <el-button type="primary" :icon="Edit" link @click="EditSpeedVisible = true" />
        </div>
        <div class="stat-value">{{ state.show.speedBit }}</div>
        <div class="stat-sub" v-if="state.maxSpeed">/ {{ formatter(state.maxSpeed, 2, [0, 0, 0, 0, 0, 0]) }}</div>
      </div>
    </div>

    <div class="ios-main-action">
      <button class="ios-btn-primary" v-if="!isRunning && !state.isChecking" @click="tryStart">
        <el-icon :size="24"><CaretRight /></el-icon>
        开始测试
      </button>
      <button class="ios-btn-loading" v-if="state.isChecking" disabled>
        <el-icon class="is-loading" :size="24"><Loading /></el-icon>
        正在准备
      </button>
      <button class="ios-btn-danger" v-if="isRunning && !state.isChecking" @click="isRunning = false">
        <el-icon :size="24"><VideoPause /></el-icon>
        停止测试
      </button>
    </div>

    <div class="ios-extra-actions">
      <el-button type="primary" :icon="FullScreen" link @click="isFullScreen = true" />
      <el-button type="primary" :icon="chartShow ? Hide : TrendCharts" link @click="chartShow = !chartShow" />
    </div>

    <div v-show="chartShow" ref="chartContainer" class="ios-chart"></div>
  </div>

  <!-- Dialogs -->
  <el-dialog v-model="EditTableVisible" title="自定义地址" width="90%" class="ios-dialog">
    <el-table :data="customNodes" style="width: 100%" max-height="300">
      <el-table-column prop="label" label="名称" />
      <el-table-column prop="value" label="URL" />
      <el-table-column width="50">
        <template #default="scope">
          <el-button type="danger" link :icon="Delete" @click="customNodes.splice(scope.$index, 1)" />
        </template>
      </el-table-column>
    </el-table>
    <template #footer>
      <el-button class="ios-btn-secondary" @click="addTableVisible = true">添加地址</el-button>
    </template>
  </el-dialog>

  <el-dialog v-model="addTableVisible" title="添加链接" width="90%" class="ios-dialog">
    <el-form :model="addForm" label-position="top">
      <el-form-item label="名称">
        <el-input v-model="addForm.label" placeholder="节点名称" />
      </el-form-item>
      <el-form-item label="URL">
        <el-input v-model="addForm.value" placeholder="https://..." />
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="addTableVisible = false">取消</el-button>
      <el-button type="primary" @click="addNode" :loading="addForm.checking">确认</el-button>
    </template>
  </el-dialog>

  <el-dialog v-model="EditMaxVisible" title="流量上限" width="300px" class="ios-dialog">
    <el-input type="number" v-model="maxUseInput.num">
      <template #append>
        <el-select v-model="maxUseInput.type" style="width: 70px">
          <el-option label="MB" value="MB" />
          <el-option label="GB" value="GB" />
        </el-select>
      </template>
    </el-input>
    <template #footer>
      <el-button type="primary" @click="editMaxUse">确定</el-button>
    </template>
  </el-dialog>

  <el-dialog v-model="EditSpeedVisible" title="带宽上限" width="300px" class="ios-dialog">
    <el-input type="number" v-model="maxSpeedInput.num">
      <template #append>
        <el-select v-model="maxSpeedInput.type" style="width: 70px">
          <el-option label="Mbps" value="Mbps" />
          <el-option label="Gbps" value="Gbps" />
        </el-select>
      </template>
    </el-input>
    <template #footer>
      <el-button type="primary" @click="editMaxSpeed">确定</el-button>
    </template>
  </el-dialog>

  <FullScreenUI v-if="isFullScreen" :isVisible="isFullScreen" :state="state" :isRunning="isRunning" @close="isFullScreen = false" />
</template>

<script setup lang="ts">
import { ref, reactive, watch, onMounted, onUnmounted, type Ref } from 'vue'
import * as echarts from 'echarts'
import { ElMessage } from 'element-plus'
import { toClipboard } from '@soerenmartius/vue3-clipboard'
import { 
  Edit, Delete, Loading, CopyDocument, TrendCharts, Hide, 
  Calendar, FullScreen, CaretRight, VideoPause 
} from '@element-plus/icons-vue'
import nodesJson from "../assets/nodes.json"
import FullScreenUI from './FullScreen.vue'

const props = defineProps({ isVisible: Boolean })

// Nodes Logic
const customNodes = reactive(localStorage.customNodes ? JSON.parse(localStorage.customNodes) : [])
const nodes: Ref<any[]> = ref([])

const updateNodes = () => {
  const online = []
  for (let groupName in nodesJson) {
    const group = nodesJson[groupName as keyof typeof nodesJson]
    const options = Object.entries(group).map(([label, value]) => ({ label, value }))
    online.push({ label: groupName, options })
  }
  nodes.value = [...online, { label: '自定义', options: customNodes }]
}
updateNodes()
watch(customNodes, () => {
  localStorage.customNodes = JSON.stringify(customNodes)
  updateNodes()
}, { deep: true })

const runUrl = ref(nodes.value[0]?.options[0]?.value || '')
const threadNum = ref(Number(localStorage.threadNum) || 16)
watch(threadNum, (n) => localStorage.threadNum = n)

const runBackground = ref(localStorage.runBackground === 'true')
watch(runBackground, (n) => localStorage.runBackground = n)
const autoStart = ref(localStorage.autoStart === 'true')
watch(autoStart, (n) => localStorage.autoStart = n)

// State
const isRunning = ref(false)
const state = reactive({
  isChecking: false,
  maxUse: Number(localStorage.maxUse) || 0,
  maxSpeed: Number(localStorage.maxSpeed) || 0,
  show: { allUsed: '0 B', speed: '0 B/s', speedBit: '0 bps' },
  predict: { min: '0 B', hour: '0 B', day: '0 B', mon: '0 B' }
})

// UI Controls
const EditTableVisible = ref(false)
const addTableVisible = ref(false)
const EditMaxVisible = ref(false)
const EditSpeedVisible = ref(false)
const chartShow = ref(false)
const isFullScreen = ref(false)
const addForm = reactive({ label: '', value: '', checking: false })
const maxUseInput = reactive({ num: '', type: 'GB' })
const maxSpeedInput = reactive({ num: '', type: 'Mbps' })

// Methods
const copyUrl = () => {
  toClipboard(runUrl.value)
  ElMessage.success('已复制测速地址')
}

const formatter = (num: number, type: number = 0, flo: number[] = [2, 2, 2, 2, 2, 2]) => {
  const units = type === 0 ? ['B', 'KB', 'MB', 'GB', 'TB'] : ['bps', 'Kbps', 'Mbps', 'Gbps', 'Tbps']
  let i = 0
  while (num >= 1024 && i < units.length - 1) {
    num /= 1024
    i++
  }
  return num.toFixed(flo[i]) + ' ' + units[i]
}

const tryStart = () => {
  if (!runUrl.value) return ElMessage.warning('请选择测速地址')
  isRunning.value = true
}

const addNode = () => {
  if (!addForm.label || !addForm.value) return
  customNodes.push({ label: addForm.label, value: addForm.value })
  addTableVisible.value = false
  addForm.label = ''
  addForm.value = ''
}

const editMaxUse = () => {
  const factor = { 'MB': 1024 * 1024, 'GB': 1024 * 1024 * 1024 }[maxUseInput.type as 'MB' | 'GB'] || 1
  state.maxUse = Number(maxUseInput.num) * factor
  localStorage.maxUse = state.maxUse
  EditMaxVisible.value = false
}

const editMaxSpeed = () => {
  const factor = { 'Mbps': 1024 * 1024 / 8, 'Gbps': 1024 * 1024 * 1024 / 8 }[maxSpeedInput.type as 'Mbps' | 'Gbps'] || 1
  state.maxSpeed = Number(maxSpeedInput.num) * factor
  localStorage.maxSpeed = state.maxSpeed
  EditSpeedVisible.value = false
}

// Chart Logic
const chartContainer = ref<HTMLElement | null>(null)
let chart: echarts.ECharts | null = null
watch(chartShow, (val) => {
  if (val && !chart) {
    setTimeout(() => {
      if (chartContainer.value) {
        chart = echarts.init(chartContainer.value)
        chart.setOption({
          grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
          xAxis: { type: 'category', boundaryGap: false, data: [] },
          yAxis: { type: 'value' },
          series: [{ data: [], type: 'line', smooth: true, areaStyle: {}, itemStyle: { color: '#007aff' } }]
        })
      }
    }, 100)
  }
})

// Mock data update for UI demo
let interval: any
onMounted(() => {
  interval = setInterval(() => {
    if (isRunning.value) {
      const mockSpeed = Math.random() * 100 * 1024 * 1024 // 100MB/s
      state.show.speed = formatter(mockSpeed) + '/s'
      state.show.speedBit = formatter(mockSpeed * 8, 1)
      // Update other stats...
    }
  }, 1000)
})
onUnmounted(() => clearInterval(interval))

</script>

<style scoped>
.ios-card {
  border-radius: var(--radius-ios-lg);
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.ios-section {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.ios-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.ios-label {
  font-size: 15px;
  font-weight: 600;
  color: var(--text-secondary);
}

.ios-value {
  font-size: 15px;
  font-weight: 600;
  color: var(--ios-blue);
}

.ios-select {
  width: 100%;
}

:deep(.el-input__wrapper) {
  background: rgba(0,0,0,0.05) !important;
  box-shadow: none !important;
  border-radius: 10px !important;
}

.ios-switches {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

.ios-switch-item {
  background: rgba(0,0,0,0.03);
  padding: 12px;
  border-radius: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
}

.ios-stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.ios-stat-box {
  background: rgba(0,0,0,0.03);
  padding: 15px 10px;
  border-radius: 16px;
  text-align: center;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.ios-stat-box.highlight {
  background: rgba(0, 122, 255, 0.1);
  border: 1px solid rgba(0, 122, 255, 0.2);
}

.stat-header {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 4px;
}

.stat-label {
  font-size: 12px;
  color: var(--text-secondary);
  font-weight: 500;
}

.stat-value {
  font-size: 18px;
  font-weight: 700;
  color: var(--text-main);
}

.stat-sub {
  font-size: 10px;
  color: var(--text-secondary);
}

.ios-main-action {
  margin-top: 10px;
}

.ios-btn-primary, .ios-btn-danger, .ios-btn-loading {
  width: 100%;
  height: 54px;
  border-radius: 16px;
  border: none;
  font-size: 17px;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  cursor: pointer;
  transition: all 0.2s;
}

.ios-btn-primary {
  background: var(--ios-blue);
  color: white;
}

.ios-btn-primary:active {
  transform: scale(0.98);
  opacity: 0.9;
}

.ios-btn-danger {
  background: #ff3b30;
  color: white;
}

.ios-btn-loading {
  background: #e5e5ea;
  color: #8e8e93;
  cursor: not-allowed;
}

.ios-extra-actions {
  display: flex;
  justify-content: center;
  gap: 20px;
}

.ios-chart {
  width: 100%;
  height: 300px;
  margin-top: 10px;
}

.predict-list {
  font-size: 13px;
  line-height: 1.6;
}

@media (max-width: 600px) {
  .ios-stats-grid {
    grid-template-columns: 1fr;
  }
  .ios-switches {
    grid-template-columns: 1fr;
  }
}

@media (prefers-color-scheme: dark) {
  .ios-stat-box, .ios-switch-item {
    background: rgba(255,255,255,0.05);
  }
  :deep(.el-input__wrapper) {
    background: rgba(255,255,255,0.1) !important;
  }
}
</style>
