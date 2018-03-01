<template>
  <div id="lineChart" ref="wrapper">
    <canvas id="canvas"
            ref="canvas"
            @mouseleave="canvasMouseLeave"
            @mousemove="canvasMouseMove">
    </canvas>
    <div class="download" v-if="load" @click="download">下载图片</div>
  </div>
</template>

<script type="text/ecmascript-6">
  export default {
    props: {
      rowData: {
        type: Array,
        default: []
      },
      colData: {
        type: Array,
        default: () => []
      },
      colStart: {
        type: Number,
        default: 1
      },
      row: {
        type: Number,
        default: 5
      },
      load: {
        type: Boolean,
        default: true
      },
      hoverLine: {
        type: Boolean,
        default: true
      },
      playing: {
        type: Boolean,
        default: true
      },
      imgType: {
        type: String,
        default: 'jpg'
      }
    },
    data () {
      return {
        Axis: []
      }
    },
    methods: {
      saveFile (rowData, filename) {
        let saveLink = document.createElementNS('http://www.w3.org/1999/xhtml', 'a')
        saveLink.href = rowData
        saveLink.download = filename
        let event = document.createEvent('MouseEvents')
        event.initMouseEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null)
        saveLink.dispatchEvent(event)
      },
      download () {
        let canvas = this.$refs.canvas
        if (!this.imgType.match(/png|jpeg|jpg|bmp|gif/)) {
          this.imgType = 'jpg'
        }
        this.saveFile(canvas.toDataURL(`image/${this.imgType}`), `${new Date().getTime()}.${this.imgType}`)
      },
      canvasMouseMove (ev) {
        ev = ev || event
        if (!this.hoverLine) {
          return
        }
        let canvas = this.$refs.canvas
        let cxt = canvas.getContext('2d')
        let disX = ev.clientX - this.$refs.wrapper.offsetLeft
        for (let i = 0; i < this.Axis.length; i++) {
          if (disX >= parseInt(this.Axis[i].x) - 2 && disX <= parseInt(this.Axis[i].x) + 2) {
            cxt.clearRect(0, 0, this.$refs.canvas.offsetWidth, this.$refs.canvas.offsetHeight)
            this.initCanvas(false)
            this.createStroke(this.Axis[i].x, 0, this.Axis[i].x, this.$refs.canvas.offsetHeight, cxt)
          }
        }
      },
      canvasMouseLeave () {
        if (!this.hoverLine) {
          return
        }
        this.initCanvas(false)
      },
      numberToFixed (num) {
        num = num.toFixed(2)
        num = (num - parseInt(num)) === 0 ? parseInt(num) : num
        return num
      },
      createCanvas () {
        clearInterval(this.timer)
        this.timer = setInterval(() => {
          this.createAxis()
          if (this.rowData.length) {
            this.initCanvas(this.playing)
            clearInterval(this.timer)
          }
        }, 30)
      },
      getNumberSize (num, min = false) {
        if (min) {
          return Math.floor(num / 5) * 5
        }
        if (num < 0 && !min) {
          return Math.ceil(num / 5) * 5
        }
        if (num < 0) {
          return Math.ceil(num / 5) * 5
        }
        if (num >= 0) {
          return Math.ceil(num / 5) * 5
        }
      },
      setRowNumber (height, cxt) {
        let row = (height - 50) / this.row
        let max = this.getNumberSize(Math.max.apply(Math, this.rowData))
        let min = this.getNumberSize(Math.min.apply(Math, this.rowData), true)
        let num = max - min
        for (let i = 0; i < this.row; i++) {
          this.createText(20, height - (47 + i * row), cxt, this.numberToFixed((min + i * num / (this.row - 1))))
        }
      },
      setColNumber (width, height, cxt) {
        for (let i = this.colStart; i < (this.rowData.length + this.colStart); i++) {
          this.createText(58 + (width - 60) / this.rowData.length * (i - this.colStart), height - 20, cxt, this.colData.length <= 0 ? i : this.colData[i - 1])
        }
      },
      setWeatherPosition (currentNumber) {
        let max = this.getNumberSize(Math.max.apply(Math, this.rowData))
        let min = this.getNumberSize(Math.min.apply(Math, this.rowData))
        let num = max - min <= 0 ? 1 : max - min
        let percen = (currentNumber - min) / num
        // (this.row - 1) * ((this.$refs.canvas.offsetHeight - 50) / this.row) 动态获取数据范围的高度
        return ((this.row - 1) * ((this.$refs.canvas.offsetHeight - 50) / this.row)) * Math.abs(percen)
      },
      createArc (x, y, context, w = 3, h = 3) {
        context.beginPath()
        context.fillStyle = '#fff'
        context.strokeStyle = '#fff'
        context.arc(x, y, w, h, Math.PI * 4)
        context.fill()
        context.stroke()
        context.closePath()
      },
      createText (x, y, context, text) {
        context.beginPath()
        context.fillStyle = '#fff'
        context.fillText(text, x, y)
        context.closePath()
      },
      createStroke (x, y, toX, toY, context) {
        context.beginPath()
        context.moveTo(x, y)
        context.lineTo(toX, toY)
        context.lineWidth = 1
        context.strokeStyle = `#fff`
        context.stroke()
        context.closePath()
      },
      createAxis () {
        this.Axis = []
        let max = this.getNumberSize(Math.max.apply(Math, this.rowData))
        let min = this.getNumberSize(Math.min.apply(Math, this.rowData))
        let num = max - min <= 0 ? 1 : max - min
        let width = this.$refs.canvas.offsetWidth
        let height = this.$refs.canvas.offsetHeight
        for (let i = 0; i < this.rowData.length; i++) {
          this.Axis.push({
            x: 60 + (width - 60) / this.rowData.length * i,
            y: num === 1 ? this.$refs.canvas.offsetHeight / 2 : height - this.setWeatherPosition(this.rowData[i]) - 50
          })
        }
      },
      initLine (cxt, play = true) {
        if (play) {
          let i = 0
          let timer = null
          clearInterval(timer)
          timer = setInterval(() => {
            this.createArc(this.Axis[i].x, this.Axis[i].y, cxt)
            this.createText(this.Axis[i].x - 5, this.Axis[i].y - 10, cxt, this.rowData[i])
            if (!this.Axis[i + 1]) {
              clearInterval(timer)
              return
            }
            this.createStroke(this.Axis[i].x, this.Axis[i].y, this.Axis[i + 1].x, this.Axis[i + 1].y, cxt)
            i++
          }, 30)
        } else {
          for (let i = 0; i < this.Axis.length; i++) {
            this.createArc(this.Axis[i].x, this.Axis[i].y, cxt)
            this.createText(this.Axis[i].x - 5, this.Axis[i].y - 10, cxt, this.rowData[i])
            if (!this.Axis[i + 1]) {
              return
            }
            this.createStroke(this.Axis[i].x, this.Axis[i].y, this.Axis[i + 1].x, this.Axis[i + 1].y, cxt)
          }
        }
      },
      initCanvas (play = true) {
        let canvas = this.$refs.canvas
        let cxt = canvas.getContext('2d')
        let width = this.$refs.canvas.offsetWidth
        let height = this.$refs.canvas.offsetHeight
        canvas.width = this.$refs.canvas.offsetWidth
        canvas.height = this.$refs.canvas.offsetHeight
        cxt.clearRect(0, 0, width, height)
        cxt.fillStyle = `rgba(0,0,0,.5)`
        cxt.fillRect(0, 0, width, height)
        this.setRowNumber(height, cxt)
        this.setColNumber(width, height, cxt)
        this.initLine(cxt, play)
      }
    },
    mounted () {
      this.createCanvas()
      window.addEventListener('resize', () => {
        this.createAxis()
        this.initCanvas(this.playing)
      })
    }
  }
</script>

<style scoped>
  #lineChart {
    width: 100%;
    height: 100%;
    position: relative;
  }
  #lineChart #canvas {
    width: 100%;
    height: 100%;
    cursor: pointer;
  }
  #lineChart .download {
    padding: 0 10px;
    height: 30px;
    line-height: 30px;
    position: absolute;
    right: 0;
    top: 0;
    cursor: pointer;
    text-align: center;
  }
  #lineChart .download:hover {
    color: #f00;
  }
</style>