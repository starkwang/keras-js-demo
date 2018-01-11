<style lang="less">
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
  margin: 20px auto;
  line-height: 30px;
  font-size: 18px;
  cursor: pointer;
}

.canvas-container {
  position: relative;
  width: 280px;
  height: 280px;
  margin: 0 auto;
  .mask {
    width: 100%;
    height: 100%;
    line-height: 280px;
    color: #fff;
    font-size: 24px;
    background: rgba(0, 0, 0, 0.5);
    position: absolute;
    top: 0;
    left: 0;
  }
}



canvas {
  border: 1px solid #DDD;
}
</style>

<template>
  <div id="app">
    <div class="canvas-container">
      <canvas
        width="280"
        height="280"
        ref="canvas"
        @mousedown="onMouseDown"
        @mousemove="onMouseMove"
        @mouseup="drawEnd"
        @mouseleave="drawEnd"
        @touchstart="onMouseDown"
        @touchmove="onMouseMove"
        @touchend="drawEnd"
      />
      <div class="mask" v-show="loading">加载模型中：{{modelLoadingProgress}}%</div>
    </div>
    <div class="clear" @click="clear">clear</div>
    <p class="result">识别结果：{{result}}</p>
  </div>
</template>

<script>
import { reduceData, getCoordinates } from './utils'
export default {
  name: 'app',
  data () {
    return {
      result: '',
      loading: true,
      modelLoadingProgress: 0
    }
  },
  created () {
    this.model = new window.KerasJS.Model({
      filepath: process.env.NODE_ENV === 'production' ? '/keras-js-demo/dist/static/mnist_cnn.bin' : '/static/mnist_cnn.bin',
      gpu: true,
      transferLayerOutputs: true
    })
    this.model.ready().then(() => {
      console.log('keras init')
      this.loading = false
    })
    this.model.events.on('loadingProgress', this.handleLoadingProgress)
  },
  methods: {
    handleLoadingProgress (progress) {
      this.modelLoadingProgress = Math.round(progress)
    },
    onMouseDown (e) {
      e.preventDefault()
      this.drawLine(...getCoordinates(e))
      this.listenMouseMove = true
    },
    onMouseMove (e) {
      e.preventDefault()
      console.log('onMouseMove')
      if (!this.listenMouseMove) return
      this.drawLine(...getCoordinates(e))
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
