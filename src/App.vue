<template>
  <h4>Record</h4>
  <!-- <h5>{{ '00 : ' + formatNum(time) }}</h5> -->
  <div class="row justify-center q-mb-xl">
    <div class="col">
      <h5>{{ state.recorder?.duration || 0 }}</h5>录制时长(秒)
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
    <q-btn color="deep-orange" glossy text-color="black" push label="Play" icon-right="play_circle"
      @click="playRecorder" />
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
  initRecorder()
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
  state.recorder?.start()
  state.timer = setInterval(() => {
    time.value++
  }, 1000)
}

function pauseRecorder(): void {
  state.recorder?.pause()
  clearInterval(state.timer!)
  state.timer = null
}

function resumeRecorder(): void {
  state.recorder?.resume()
  state.timer = setInterval(() => {
    time.value++
  }, 1000)
}

function stopRecorder(): void {
  state.recorder?.stop()
  pauseRecorder()
  time.value = 0
}

function playRecorder(): void {

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
  margin: 0
h6
  margin: 0 0 25px
</style>
