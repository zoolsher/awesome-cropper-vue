<template>
  <canvas width="500px" height="500px"></canvas>
</template>
<script>
  import platform from 'platform'
  const LONG_TIRM = 1000
  export default {
    name: 'awesome-cropper',
    data () {
      return {
        originalImageHeight: 0,
        originalImageWidth: 0,
        canvasWidth: 0,
        canvasHeight: 0,
        moveX: 0,
        moveY: 0,
        scale: 1,
        minScale: 1,
        offsetX: 0,
        offsetY: 0,
        lastMove: new Date().getTime(),
        cropX: 20.5,
        cropY: 20.5,
        cropWidth: 200,
        cropHeight: 200,
        cropMax: 0,
        responseEvent: true,
        contactingDirection: 0, // 0 for x ,1 for y
        src: 'http://warehouse.squarance.com/0001-avant-garde-6-2-yaohua-wang.jpg'
      }
    },
    methods: {
      onFrameUpdate: function () {
        this.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)
        this.ctx.setTransform(this.scale || 1, 0, 0, this.scale || 1, this.moveX, this.moveY)
        this.drawImage()
        this.drawCrop()
        window.requestAnimationFrame(this.onFrameUpdate)
      },
      drawCrop: function () {
        this.ctx.setTransform(1, 0, 0, 1, 0, 0)
        this.ctx.lineWidth = 3
        this.ctx.strokeStyle = 'green'
        this.ctx.strokeRect(this.cropX, this.cropY, this.cropWidth, this.cropHeight)
      },
      drawImage: function () {
        if (this.image) {
          this.ctx.drawImage(this.image, 0, 0)
        }
      },
      dragStart: function (evt) {
        if (!this.responseEvent) {
          return
        }
        this.updateLastMove()
        this.addCheckTimer()
        this.isdrag = true
        this.last_x = evt.x
        this.last_y = evt.y
      },
      removeDragEvent: function (evt) {
        this.updateLastMove()
        this.addCheckTimer()
        this.isdrag = false
      },
      dragMove: function (evt) {
        if (!this.responseEvent) {
          return
        }
        if (this.isdrag) {
          this.updateLastMove()
          this.addCheckTimer()
          if (platform.name !== 'Safari') {
            this.moveX += evt.movementX
            this.moveY += evt.movementY
          } else {
            this.moveX += evt.x - this.last_x
            this.moveY += evt.y - this.last_y
            this.last_x = evt.x
            this.last_y = evt.y
          }
        }
        evt.preventDefault()
      },
      scalePic: function (evt) {
        if (!this.responseEvent) {
          return
        }
        this.updateLastMove()
        this.addCheckTimer()
        let deltaY = evt.wheelDelta
        evt.preventDefault()
        var targetScale = this.deltaToScale(deltaY)
        if (targetScale < this.minScale) {
          targetScale = this.minScale
        }
        this.moveX = (this.moveX - evt.offsetX) * (targetScale / this.scale) + evt.offsetX
        this.moveY = (this.moveY - evt.offsetY) * (targetScale / this.scale) + evt.offsetY
        this.changeScale(deltaY)
      },
      deltaToScale: function (delta) {
        return this.scale + delta / 200
      },
      changeScale: function (delta) {
        this.scale = this.deltaToScale(delta)
        if (this.scale < this.minScale) {
          this.scale = this.minScale
        }
      },
      animateToNormal: function () {
        var newMoveX = this.moveX
        if (this.moveX > this.cropX) {
          newMoveX = this.cropX
        }
        var newMoveY = this.moveY
        if (this.moveY > this.cropY) {
          newMoveY = this.cropY
        }
        var imageMaxX = this.moveX + (this.originalImageWidth * this.scale)
        var imageMaxY = this.moveY + (this.originalImageHeight * this.scale)
        var cropMaxX = this.cropX + this.cropWidth
        var cropMaxY = this.cropY + this.cropHeight
        var newScale = this.scale
        if (imageMaxX < cropMaxX) {
          newMoveX += (cropMaxX - imageMaxX)
        }
        if (imageMaxY < cropMaxY) {
          newMoveY += (cropMaxY - imageMaxY)
        }
        this.animate(newMoveX, newMoveY, newScale)
      },
      animate: function (moveX, moveY, scale) {
        var length = 100
        var counter = 0
        var startX = this.moveX
        var startY = this.moveY
        var startScale = this.scale
        var that = this
        var timer = setInterval(function () {
          counter++
          if (that.moveX !== moveX) {
            that.moveX = that.easeInOut(counter / length, startX, moveX - startX)
          }
          if (that.moveY !== moveY) {
            that.moveY = that.easeInOut(counter / length, startY, moveY - startY)
          }
          if (that.scale !== scale) {
            that.scale = that.easeInOut(counter / length, startScale, scale - startScale)
          }
          if (counter > length) {
            clearInterval(timer)
            that.responseEvent = true
          }
        }, 10)
      },
      updateLastMove: function () {
        this.lastMove = new Date().getTime()
      },
      longTimeNoMove: function () {
        this.lastMove = new Date().getTime()
        this.responseEvent = false
        this.animateToNormal()
        this.clearCheckTimer()
      },
      checkIfMove: function (evt) {
        if (!this.responseEvent) {
          return
        }
        var now = new Date().getTime()
        if (now - this.lastMove > LONG_TIRM) {
          this.longTimeNoMove()
        }
      },
      addCheckTimer: function () {
        if (this.timerr > 0) {
          return
        }
        this.timerr = setInterval(this.checkIfMove, 1000)
      },
      clearCheckTimer: function () {
        if (this.timerr > 0) {
          clearInterval(this.timerr)
          this.timerr = 0
        }
      },
      easeIn: function (t, start, end) {
        return end * Math.pow(t, 3) + start
      },
      easeOut: function (t, start, end) {
        return end * (Math.pow(t - 1, 3) + 1) + start
      },
      easeInOut: function (t, start, end) {
        if (t * 2 < 1) {
          return end * Math.pow(t * 2, 3) / 2 + start
        } else {
          return end * (Math.pow(t * 2 - 2, 3) + 2) / 2 + start
        }
      }
    },
    mounted () {
      window.requestAnimationFrame(this.onFrameUpdate)
      let image = new Image()
      let that = this
      this.canvasWidth = this.$el.width
      this.canvasHeight = this.$el.height
      image.onload = function () {
        that.originalImageHeight = this.height
        that.originalImageWidth = this.width
        var scaleY = that.canvasHeight / this.height
        var scaleX = that.canvasWidth / this.width
        if (scaleX > scaleY) {
          that.scale = scaleY
          that.contactingDirection = 1
          that.cropMax = this.width / that.scale
        } else {
          that.scale = scaleX
          that.contactingDirection = 0
          that.cropMax = this.height / that.scale
        }
        that.minScale = that.scale
        that.image = this
      }
      image.src = this.src
      this.ctx = this.$el.getContext('2d')
      this.$el.addEventListener('mousemove', this.dragMove)
      this.$el.addEventListener('mouseleave', this.removeDragEvent)
      this.$el.addEventListener('mouseup', this.removeDragEvent)
      this.$el.addEventListener('mousedown', this.dragStart)
      this.$el.addEventListener('mousewheel', this.scalePic)
    }
  }
</script>
