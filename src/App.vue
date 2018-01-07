<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.clear {
  background: #2c3e50;
  color: #fff;
  border-radius: 4px;
  width: 100px;
  margin: 0 auto;
  line-height: 30px;
  font-size: 18px;
  cursor: pointer;
}

canvas {
  border: 1px solid #DDD;
}
</style>

<template>
  <div id="app">
    <canvas
      width="280"
      height="280"
      ref="canvas"
      @mousedown="onMouseDown"
      @mousemove="onMouseMove"
      @mouseup="drawEnd"
      @mouseleave="drawEnd"
    />
    <div class="clear" @click="clear">clear</div>
    <p class="result">识别结果：{{result}}</p>
  </div>
</template>

<script>
import { reduceData } from './utils'
export default {
  name: 'app',
  data () {
    return {
      result: ''
    }
  },
  created () {
    this.model = new window.KerasJS.Model({
      filepath: '/static/mnist_cnn.bin',
      gpu: true,
      transferLayerOutputs: true
    })
    this.model.ready().then(() => console.log('keras init'))
  },
  methods: {
    onMouseDown (e) {
      this.drawLine(e.offsetX, e.offsetY)
      this.listenMouseMove = true
    },
    onMouseMove (e) {
      if (!this.listenMouseMove) return
      this.drawLine(e.offsetX, e.offsetY)
    },
    drawEnd () {
      if (!this.listenMouseMove) return
      this.listenMouseMove = false
      delete this.previousX
      delete this.previousY
      this.predict()
    },
    drawLine (x, y) {
      const { canvas } = this.$refs
      const ctx = canvas.getContext('2d')
      ctx.lineWidth = 20
      ctx.lineJoin = ctx.lineCap = 'round'
      ctx.strokeStyle = '#393E46'
      ctx.beginPath()
      const { previousX, previousY } = this
      if (previousX !== undefined) {
        ctx.moveTo(previousX, previousY)
      } else {
        ctx.moveTo(x, y)
      }
      ctx.lineTo(x, y)
      ctx.stroke()
      this.previousX = x
      this.previousY = y
    },
    predict () {
      const { canvas } = this.$refs
      const ctx = canvas.getContext('2d')
      const data = reduceData(ctx.getImageData(0, 0, 280, 280).data)
      const inputData = {
        input: new Float32Array(data)
      }
      this.model.predict(inputData).then(outputData => {
        var max = outputData.output.reduce((a, b) => Math.max(a, b), 0)
        console.log(outputData.output.indexOf(max))
        this.result = outputData.output.indexOf(max)
      })
    },
    clear () {
      const { canvas } = this.$refs
      const ctx = canvas.getContext('2d')
      ctx.clearRect(0, 0, 280, 280)
      this.result = ''
    }
  }
}
</script>
