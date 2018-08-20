<template>
  <div>
    <canvas class="full-width" id="gauge"></canvas>
    <h2 class="text-center">{{playNote}}<sup>{{playScope}}</sup></h2>

    <div class="row">
      <div class="col-4 text-center">音量: {{playVol}}</div>
      <div class="col-4 text-center">偏差: {{playOffset}}</div>
      <div class="col-4 text-center">频率: {{playFreq}}</div>
    </div>
    <div class="fixed-bottom">
      <q-btn flat small @click="showVersion">关于</q-btn>
    </div>
  </div>
</template>

<script>
import pitchfinder from 'pitchfinder'
import { Gauge } from 'gaugeJS'
export default {
  data () {
    return {
      playFreq: '',
      playVol: 0,
      detectPitch: pitchfinder.YIN({sampleRate: 8000}),
      gauge: null,
      version: ''
    }
  },
  computed: {
    playScope: function () {
      if (this.playFreq) {
        let noteNum = 12 * (Math.log(this.playFreq / 440) / Math.log(2))
        return Math.floor((Math.round(noteNum) + 57) / 12)
      } else {
        return ''
      }
    },
    playNote: function () {
      if (this.playFreq) {
        const noteStrings = ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B']
        let noteNum = 12 * (Math.log(this.playFreq / 440) / Math.log(2))
        return noteStrings[(Math.round(noteNum) + 69) % 12]
      } else {
        return ''
      }
    },
    playOffset: function () {
      if (this.playFreq) {
        let noteNum = Math.round(12 * (Math.log(this.playFreq / 440) / Math.log(2)))
        let standFreq = 440 * Math.pow(2, noteNum / 12)
        return Math.round(300 * Math.log(this.playFreq / standFreq) / Math.log(2))
      } else {
        return ''
      }
    }
  },
  mounted () {
    cordova.getAppVersion.getVersionNumber((version) => {
      this.version = version
    })
    window.audioinput.start({
      sampleRate: 8000,
      bufferSize: 800
    })
    window.addEventListener('audioinput', this.onAudioInput, false)
    this.gauge = new Gauge(document.getElementById('gauge')).setOptions({
      angle: 0,
      lineWidth: 0.2,
      colorStop: '#7986cb',
      pointer: {
        color: '#7986cb',
        length: 0.5,
        strokeWidth: 0.02
      },
      staticLabels: {
        labels: [-20, -10, -5, 0, 5, 10, 20]
      }
    })
    this.gauge.maxValue = 20
    this.gauge.setMinValue(-20)
    this.gauge.set(0)

    window.StatusBar.hide()
    window.screen.orientation.lock('portrait')
  },
  beforeDestroy () {
    window.removeEventListener('audioinput', this.onAudioInput, false)
  },
  methods: {
    onAudioInput: function (evt) {
      let total = 0
      for (var i = 0; i < evt.data.length; i++) {
        total += Math.abs(evt.data[i])
      }
      this.playVol = Math.floor(total / evt.data.length * 100)
      if (this.playVol >= 3) {
        let freq = this.detectPitch(evt.data)
        if (freq !== null && freq >= 20 && freq <= 3200) {
          this.playFreq = Math.round(freq)
        }
        this.gauge.set(this.playOffset)
      }
    },
    showVersion: function () {
      alert(this.version)
    }
  }
}
</script>
