<template>
    <div ref="elment" v-if="modelValue" class="fullscreen"
        @click="() => emit('update:modelValue', false)"
        @wheel="(e) => e.preventDefault()"
        @touchmove="(e) => e.preventDefault()"
        >
        <div class="content">
            <div class="left">
                <div class="title">
                    <span>NetworkPanel</span>
                </div>
                <div class="time">
                    <span>{{ time }}</span>
                </div>
                <div class="date">
                    <span>{{ date }}</span>
                </div>
            </div>
            <div class="right">
                <div class="state">
                    <span class="des">总流量</span><br>
                    <span class="value">{{ state.show.allUsed }}</span><br><br>
                    <span class="des">{{ isRunning?"实时速度":"平均速度" }}</span><br>
                    <span class="value">{{ state.show.speed }}</span><br><br>
                    <span class="des">{{ isRunning?"实时带宽":"平均带宽" }}</span><br>
                    <span class="value">{{ state.show.speedBit }}</span><br><br>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import {ref,watch,onMounted, defineModel, watchEffect, onUnmounted} from 'vue'
import NoSleep from 'nosleep.js';
const props=defineProps({
  state: { type: Object, required: true },
  isRunning: { type: Boolean, required: true },
  modelValue: { type: Boolean, required: true },
})
const emit = defineEmits<{
  (e: 'update:modelValue', value: Boolean): void
}>()

const elment=ref()
document.addEventListener("fullscreenchange", function (e) {
  if (document.fullscreenElement) {
    emit('update:modelValue', true)
  } else {
    emit('update:modelValue', false)
  }
})
const isMiuiBrowser = /MiuiBrowser/i.test(navigator.userAgent)
let noSleep = isMiuiBrowser?null:new NoSleep();
watchEffect(
  async() => {
    if(props.modelValue){
        noSleep?.enable();
        if(!elment.value)return
        if(elment.value.requestFullscreen)
            elment.value.requestFullscreen()
        else if(elment.value.webkitRequestFullscreen)
            elment.value.webkitRequestFullscreen()
        else if(elment.value.mozRequestFullscreen)
            elment.value.mozRequestFullscreen()
        else if(elment.value.msRequestFullscreen)
            elment.value.msRequestFullscreen()
    }else{
        noSleep?.disable();
    }
  }
)
const time=ref("")
const date=ref("")
const dayToWeek = ['周日', '周一', '周二', '周三', '周四', '周五', '周六']
let task:number=0
onMounted(()=>{
    task=setInterval(()=>{
        const datetime=new Date()
        time.value=`${datetime.getHours().toString().padStart(2, '0')}:${datetime.getMinutes().toString().padStart(2, '0')}`
        date.value=`${datetime.getFullYear()}-${datetime.getMonth()+1}-${datetime.getDate()} ${dayToWeek[datetime.getDay()]}`
    },1000)
})
onUnmounted(()=>{
    clearInterval(task)
})
</script>
<style scoped>
:root {
 --glass-bg-light: rgba(255, 255, 255, 0.15);
 --glass-bg-dark: rgba(30, 30, 30, 0.4);
 --glass-border-light: rgba(255, 255, 255, 0.2);
 --glass-border-dark: rgba(255, 255, 255, 0.1);
 --blur-amount: 20px;
}

@font-face {
   font-family: 'DingTalk';
   src: url('../assets/DingTalk-simple.ttf') format('truetype');
}

.fullscreen {
   position: fixed;
   top: 0;
   left: 0;
   width: 100%;
   height: 100%;
   background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
   color: rgb(255, 255, 255);
   z-index: 9999999999;
   text-align: center;
}

.title {
   margin-top: 3vh;
   font-size: 32px;
   font-weight: 700;
   text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

.right {
   margin-top: 20px;
}

@media screen and (min-width: 500px) {
   .content{
       column-count: 2;
   }
   .left {
       height: 200px;

   }
   .right {
       margin-top: -1vh;
       height: 200px;
   }
}

.date {
   margin-top: -5px;
   font-size: 22px;
   opacity: 0.9;
}

.time {
   font-size: 70px;
   font-weight: 700;
   text-shadow: 0 4px 20px rgba(102, 126, 234, 0.4);
}

.state > .des {
   font-size: 16px;
   font-weight: 600;
   opacity: 0.8;
   margin-bottom: 5px;
}

.state > .value {
   font-size: 28px;
   font-weight: 700;
   color: #a78bfa;
   text-shadow: 0 2px 10px rgba(167, 139, 250, 0.4);
}

.content{
   position: absolute;
   width: 99%;
   max-width: 600px;
   left: 50%;
   top:50%;
   transform: translate(-50%, -50%);
   animation:standby 60s infinite alternate ease-in-out;
   font-family: DingTalk;
   background: var(--glass-bg-dark);
   backdrop-filter: blur(var(--blur-amount));
   -webkit-backdrop-filter: blur(var(--blur-amount));
   border: 1px solid var(--glass-border-dark);
   border-radius: 30px;
   padding: 30px;
   box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.3);
}

@keyframes standby {
   0% {
       top: 50%;
       left: 50%;
   }
   49% {
       top: 50%;
       left: 50%;
   }
   51% {
       top: 55%;
       left: 51%;
   }
   100% {
       top: 55%;
       left: 51%;
   }
}
</style>