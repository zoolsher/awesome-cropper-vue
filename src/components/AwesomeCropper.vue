import Vue from 'vue'
<template>
  <canvas width="500px" height="500px"></canvas>
</template>
<script>
  import platform from 'platform'
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
        src: 'http://square-shit.com/static/image/cover_3.jpg'
      }
    },
    methods: {
      onFrameUpdate: function () {
        console.log(this.moveX)
        window.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)
        window.ctx.setTransform(this.scale || 1, 0, 0, this.scale || 1, this.moveX, this.moveY)
        this.drawImage()
        window.requestAnimationFrame(this.onFrameUpdate)
      },
      drawImage: function () {
        if (this.image) {
          window.ctx.drawImage(this.image, 0, 0)
        }
      },
      dragStart: function (evt) {
        this.isdrag = true
        this.last_x = evt.x
        this.last_y = evt.y
      },
      removeDragEvent: function (evt) {
        this.isdrag = false
      },
      dragMove: function (evt) {
        if (this.isdrag) {
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
        let deltaY = evt.wheelDelta
        evt.preventDefault()
        this.moveX = -evt.offsetX * (this.scale - 1)
        this.moveY = -evt.offsetY * (this.scale - 1)
        this.changeScale(deltaY)
      },
      changeScale: function (delta) {
        this.scale += delta / 100
        if (this.scale < this.minScale) {
          this.scale = this.minScale
        }
      }
    },
    mounted () {
      window.requestAnimationFrame(this.onFrameUpdate)
      let image = new Image()
      let that = this
      image.onload = function () {
        that.originalImageHeight = this.height
        that.originalImageWidth = this.width
        that.image = this
      }
      image.src = this.src
      this.canvasWidth = this.$el.width
      this.canvasHeight = this.$el.height
      window.ctx = this.$el.getContext('2d')
      this.$el.addEventListener('mousemove', this.dragMove)
      this.$el.addEventListener('mouseleave', this.removeDragEvent)
      this.$el.addEventListener('mouseup', this.removeDragEvent)
      this.$el.addEventListener('mousedown', this.dragStart)
      this.$el.addEventListener('mousewheel', this.scalePic)
    }
  }
</script>
