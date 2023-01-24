<template>
    <!-- Record -->
    <h4>Record</h4>
    <div class="row justify-center">
        <div class="col">
            <h5>{{ formatSecond(state.recorder?.duration || 0) || 0 }}</h5>录制时长(秒)
        </div>
        <div class="col">
            <h5>{{ state.recorder?.fileSize || 0 }}</h5>录音大小(字节)
        </div>
    </div>
    <canvas id="wave" width="800" height="200" class="canvas">您的浏览器不支持 HTML5 canvas 标签。</canvas>
    <!-- <div id="waveform"></div> -->
    <q-btn-group rounded>
        <q-btn color="yellow" glossy text-color="black" push label="Record" icon="fiber_manual_record"
            @click="startRecorder" />
        <q-btn color="amber" glossy text-color="black" push label="Pause" icon="pause" @click="pauseRecorder" />
        <q-btn color="amber" glossy text-color="black" push label="Resume" icon="play_arrow" @click="resumeRecorder" />
        <q-btn color="orange" glossy text-color="black" push label="Stop" icon="stop" @click="stopRecorder" />
        <q-btn color="orange" glossy text-color="black" push label="Reset" icon-right="replay" @click="resetRecorder" />
    </q-btn-group>
    <q-dialog v-model="alert">
        <q-card class="bg-dark">
            <q-card-section>
                <div class="text-h6">Alert</div>
            </q-card-section>
            <q-card-section class="q-pt-none" style="min-width: 300px;">录制时长不能超过{{ MAX_TIME }}秒</q-card-section>
            <q-card-actions align="right">
                <q-btn flat label="OK" color="primary" v-close-popup="-1" />
            </q-card-actions>
        </q-card>
    </q-dialog>
    <!-- Play -->
    <h4>Play</h4>
    <h5>{{ '00 : ' + formatNum(time) }}</h5>
    <q-btn-group rounded>
        <q-btn color="deep-orange" glossy text-color="black" push label="Play" icon-right="play_circle"
            @click="startPlayer" />
    </q-btn-group>
    <!-- Download -->
    <h4>Download</h4>
    <q-btn-group push>
        <q-btn color="blue" glossy text-color="black" push label="PCM" icon="file_download" />
        <q-btn color="blue" glossy text-color="black" push label="WAV" />
    </q-btn-group>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import Recorder from 'js-audio-recorder'
import WaveSurfer from 'wavesurfer.js'

interface State {
    timer: NodeJS.Timeout | null,
    recorder: Recorder | null,
    wavesurfer: any
}

const MAX_TIME = 10
const time = ref<number>(0)
const alert = ref<boolean>(false)
const state: State = reactive({
    timer: null,
    recorder: null,
    wavesurfer: null
})

onMounted(() => {
    state.timer = null
    state.recorder = null
    initWaveform()
})

// https://recorder-api.zhuyuntao.cn/
function initRecorder(): void {
    state.recorder = new Recorder({
        sampleBits: 16,
        sampleRate: 16000,
        numChannels: 1,
    })
    state.recorder.onprogress = function (params) {
        if (params.duration > MAX_TIME) {
            stopRecorder()
            alert.value = true
        }
        console.log(state.recorder?.getRecordAnalyseData())
    }
}

// https://wavesurfer-js.org/
function initWaveSurfer(): void {
    state.wavesurfer = WaveSurfer.create({
        container: '#waveform',
        waveColor: 'violet',
        progressColor: 'purple'
    })
}

function initWaveform(): void {
    let element = document.getElementById('wave')
    let ctx = (element! as any).getContext('2d')
    let grad = ctx.createLinearGradient(-500, -150, -500, 150)
    ctx.translate(500, 150)
    grad.addColorStop(0, "red")
    grad.addColorStop(0.5, "blue")
    grad.addColorStop(1, "yellow")
    ctx.lineWidth = 1

    function init(val) {
        ctx.clearRect(-400, -150, 800, 200)
        ctx.beginPath()
        let m = 10
        let n = 3
        let len = 100
        let ran = 16
        ran = Math.random() * 100
        for (let i = len; i > 0; i--) {
            let x = -(i * m)
            let y1 = (ran - i / 2) * n
            let y2 = -(ran - i / 2) * n

            if (y1 < 0) y1 = 0
            if (y2 > 0) y2 = 0
            if (y1 > len) y1 = len
            if (y2 < -len) y2 = -len

            ctx.moveTo(x, y1);
            ctx.lineTo(x, y2);
        }
        for (let i = 0; i < len; i++) {
            let x = i * m
            let y1 = (ran - i / 2) * n
            let y2 = -(ran - i / 2) * n
            if (y1 < 0) y1 = 0
            if (y2 > 0) y2 = 0
            if (y1 > len) y1 = len
            if (y2 < -len) y2 = -len
            ctx.moveTo(x, y1);
            ctx.lineTo(x, y2);
        }
    }
    init(4)
    ctx.strokeStyle = grad
    ctx.stroke()
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
.canvas
    margin: 20px 0
</style>
