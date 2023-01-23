<template>
  <h4>Player</h4>
  <h6>{{ '00 : ' + formatNum(time) }}</h6>
  <q-btn-group rounded>
    <q-btn color="yellow" glossy text-color="black" push label="Record" icon="fiber_manual_record"
      @click="startRecorder" />
    <q-btn color="amber" glossy text-color="black" push label="Pause" icon="pause" @click="pauseRecorder" />
    <q-btn color="orange" glossy text-color="black" push label="Stop" icon="stop" @click="stopRecorder" />
    <q-btn color="deep-orange" glossy text-color="black" push label="Play" icon-right="play_arrow"
      @click="playRecorder" />
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
  recorder: null
})

onMounted(() => {
  state.timer = null
  state.recorder = null
  initRecorder()
})

function initRecorder(): void {
  state.recorder = new Recorder({
    sampleBits: 16,     // 采样位数，支持 8 或 16，默认是16
    sampleRate: 16000,  // 采样率，支持 11025、16000、22050、24000、44100、48000，根据浏览器默认值，我的chrome是48000
    numChannels: 1,     // 声道，支持 1 或 2， 默认是1})
  })
}

function startRecorder(): void {
  state.timer = setInterval(() => {
    time.value++
  }, 1000)
}

function pauseRecorder(): void {
  clearInterval(state.timer!)
  state.timer = null
}

function stopRecorder(): void {
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

</style>
