<template>
  <h4>Record</h4>
  <div class="row justify-center q-mb-xl">
    <div class="col">
      <h5>{{ formatSecond(state.recorder?.duration || 0) || 0 }}</h5>录制时长(秒)
    </div>
    <div class="col">
      <h5>{{ state.recorder?.fileSize || 0 }}</h5>录音大小(字节)
    </div>
  </div>
  <q-btn-group rounded>
    <q-btn color="yellow" glossy text-color="black" push label="Record" icon="fiber_manual_record"
      @click="startRecorder" />
    <q-btn color="amber" glossy text-color="black" push label="Pause" icon="pause" @click="pauseRecorder" />
    <q-btn color="amber" glossy text-color="black" push label="Resume" icon="play_arrow" @click="resumeRecorder" />
    <q-btn color="orange" glossy text-color="black" push label="Stop" icon="stop" @click="stopRecorder" />
    <q-btn color="orange" glossy text-color="black" push label="Reset" icon-right="replay" @click="resetRecorder" />
  </q-btn-group>
  <h4>Play</h4>
  <h5>{{ '00 : ' + formatNum(time) }}</h5>
  <q-btn-group rounded>
    <q-btn color="deep-orange" glossy text-color="black" push label="Play" icon-right="play_circle"
      @click="startPlayer" />
  </q-btn-group>
  <h4>Download</h4>
  <q-btn-group push>
    <q-btn color="blue" glossy text-color="black" push label="PCM" icon="file_download" />
    <q-btn color="blue" glossy text-color="black" push label="WAV" />
  </q-btn-group>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import Recorder from 'js-audio-recorder'

interface State {
  timer: NodeJS.Timeout | null,
  recorder: Recorder | null
}

const time = ref<number>(0)
const state: State = reactive({
  timer: null,
  recorder: null,
})

onMounted(() => {
  state.timer = null
  state.recorder = null
})

// https://recorder-api.zhuyuntao.cn/
function initRecorder(): void {
  state.recorder = new Recorder({
    sampleBits: 16,
    sampleRate: 16000,
    numChannels: 1,
  })
}

function startRecorder(): void {
  if (state.recorder == null) {
    initRecorder()
  }
  state.recorder?.start()
}

function pauseRecorder(): void {
  state.recorder?.pause()
}

function resumeRecorder(): void {
  state.recorder?.resume()
}

function stopRecorder(): void {
  state.recorder?.stop()
}

function resetRecorder(): void {
  state.recorder?.destroy().then(() => {
    state.recorder = null
  })
}

function startPlayer(): void {
  state.recorder?.play()
  state.timer = setInterval(() => {
    time.value++
  }, 1000)
}

function pausePlayer(): void {
  state.recorder?.pausePlay()
  clearInterval(state.timer!)
  state.timer = null
}

function resumePlayer(): void {
  state.recorder?.resumePlay()
  state.timer = setInterval(() => {
    time.value++
  }, 1000)
}

function stopPlayer(): void {
  state.recorder?.stopPlay()
  clearInterval(state.timer!)
  state.timer = null
  time.value = 0
}

function formatSecond(time: number): string {
  if (!time) return ''
  let str = time.toString()
  if (str.length < 4) {
    for (let i = 0; i < 4 - str.length; i++) {
      str += '0'
    }
    return str
  } else {
    return str.slice(0, 4)
  }
}

function formatNum(num: number): string {
  if (0 < num && num < 10) return '0' + num
  if (num >= 10) return num.toString()
  return '00'
}
</script>

<style lang="sass" scoped>
h4
  margin: 40px 0
h5
  margin: 0 0 20px
</style>
