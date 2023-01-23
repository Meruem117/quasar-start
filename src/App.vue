<template>
  <h4>Player</h4>
  <h6>{{ '00 : ' + formatNum(time) }}</h6>
  <q-btn-group rounded>
    <q-btn color="yellow" glossy text-color="black" push label="Record" icon="fiber_manual_record"
      @click="startRecord" />
    <q-btn color="amber" glossy text-color="black" push label="Pause" icon="pause" @click="pauseRecord" />
    <q-btn color="orange" glossy text-color="black" push label="Stop" icon="stop" @click="stopRecord" />
    <q-btn color="deep-orange" glossy text-color="black" push label="Play" icon-right="play_arrow"
      @click="playRecord" />
  </q-btn-group>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'

const time = ref<number>(0)
const state: {
  timer: any
} = reactive({
  timer: null
})

function startRecord(): void {
  state.timer = setInterval(() => {
    time.value++
  }, 1000)
}

function pauseRecord(): void {
  clearInterval(state.timer)
  state.timer = null
}

function stopRecord(): void {
  pauseRecord()
  time.value = 0
}

function playRecord(): void {

}

function formatNum(num: number): string {
  if (0 < num && num < 10) return '0' + num
  if (num >= 10) return num.toString()
  return '00'
}
</script>

<style lang="sass" scoped>

</style>
