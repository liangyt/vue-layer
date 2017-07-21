<template>
<div class="hsy-dialog">
  <div class="mask" :style="{backgroundColor: maskColor}" @click="maskClicked"></div>
  <div class="main">
    <div class="inner">
      <slot>
        <div class="spinner">
          <div class="spinner-container container1">
            <div class="circle1"></div>
            <div class="circle2"></div>
            <div class="circle3"></div>
            <div class="circle4"></div>
          </div>
          <div class="spinner-container container2">
            <div class="circle1"></div>
            <div class="circle2"></div>
            <div class="circle3"></div>
            <div class="circle4"></div>
          </div>
          <div class="spinner-container container3">
            <div class="circle1"></div>
            <div class="circle2"></div>
            <div class="circle3"></div>
            <div class="circle4"></div>
          </div>
        </div>
      </slot>
    </div>
  </div>
</div>
</template>

<script>
const ANIME_EVENTS = ['webkitAnimationEnd', 'mozAnimationEnd', 'MSAnimationEnd', 'oanimationend', 'animationend']

export default {
  name: 'HsyDialog',

  data() {
    return {
      width: '',
      height: '',
      parentOverflowX: '',
      parentOverflowY: '',
      isHidden: true
    }
  },
  props: {
    // 自定义蒙层颜色
    maskColor: {
      type: String,
      default: 'rgba(0, 0, 0, 0.5)'
    },
    // 点击蒙层是否消失，默认不消失
    clickMask2Close: {
      type: Boolean,
      default: false
    },
    // 是否充满整个屏幕，默认充满
    wholeWindow: {
      type: Boolean,
      default: true
    },
    // 层显示/消失控制，默认不显示
    value: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    mainEl() {
      return this.$el.querySelector('.main')
    },
    // 获取父元素
    parentEl() {
      // 首先直接获取组件的父元素
      let parent = this.$el.parentNode
      while (parent) {
        // 如果父元素为body 则直接返回
        if (parent === document.body) {
          break
        }
        // 获取父元素的样式，包括style文件定义的内容
        let style = window.getComputedStyle(parent)
        // 如果父元素的position为relative或者absolute，则返回，否则一直循环直到父元素为body
        let position = style.getPropertyValue('position')
        if (position === 'relative' || position === 'absolute') {
          break
        }
        parent = parent.parentNode
      }
      return parent
    },
    // 获取父元素宽高
    parentSize: {
      cache: false,
      get() {
        // 如果是整个窗口，则直接取可显窗口的宽高
        if (this.wholeWindow) {
          let width = document.documentElement.clientWidth
          let height = document.documentElement.clientHeight
          return {
            width,
            height
          }
        }
        // 否则返回父元素的宽高
        return this.parentEl.getBoundingClientRect()
      }
    }
  },
  watch: {
    // 监控visible 控制显示/不显示
    value(val) {
      if (val) {
        this.show()
      } else {
        this.hide()
      }
    }
  },
  methods: {
    // 返回参数DOM的DOMRect
    rect(el) {
      // 如果DOMRect的width 或者 height 不为0，则直接返回该DOMRect
      let rect = el.getBoundingClientRect()

      if (rect.width !== 0 || rect.height !== 0) {
        return rect
      }
      // 如果DOMRect的width和height都为0，取回该DOM的样式
      let style = window.getComputedStyle(el)

      let display = style.getPropertyValue('display')
      let top = style.getPropertyValue('top')
      let left = style.getPropertyValue('left')
      // 组装出一个DOMRect
      el.style.top = '-1000px'
      el.style.left = '-1000px'
      el.style.display = 'inline-block'
      rect = el.getBoundingClientRect()
      // 复原该DOM的样式
      el.style.display = display
      el.style.top = top
      el.style.left = left

      return rect
    },
    // 根据父元素的宽高 设置蒙层的宽高
    updateContainerSize() {
      let size = this.parentSize
      this.$el.style.width = size.width + 'px'
      this.$el.style.height = size.height + 'px'
      this.$el.style.top = window.scrollY + 'px'
    },
    // 设置内容区域的宽
    updateMainSize() {
      let mainEl = this.$el.querySelector('.main')
      let mainElRect = this.rect(mainEl)
      mainEl.style.width = mainElRect.width + 'px'
      mainEl.style.left = '0px'
      mainEl.style.right = '0px'
    },
    // 获取父元素的 overflow
    captureParentOverflow() {
      let style = window.getComputedStyle(this.parentEl)
      this.parentOverflowX = style.getPropertyValue('overflowX')
      this.parentOverflowY = style.getPropertyValue('overflowY')
    },
    // 把父元素的overflow 设置为hidden，不让滚动
    forceParentOverflow() {
      this.parentEl.style.overflow = 'hidden'
    },
    // 恢复父元素的overflow
    resumeParentOverflow() {
      this.parentEl.style.overflowX = this.parentOverflowX
      this.parentEl.style.overflowY = this.parentOverflowY
    },
    // 给元素el 添加 class cls
    addCssClass(el, cls) {
      let clsList = el.className.split(' ')
      if (clsList.indexOf(cls) === -1) {
        clsList.push(cls)
        el.className = clsList.join(' ')
      }
    },
    // 给元素el 移除 class cls
    removeCssClass(el, cls) {
      let clsList = el.className.split(' ')
      if (clsList.indexOf(cls) !== -1) {
        el.className = clsList.filter((c) => {
          return c !== cls
        }).join(' ')
      }
    },
    // 添加动画
    anime(el, cls) {
      return new Promise((resolve) => {
        ANIME_EVENTS.forEach((e) => {
          el.addEventListener(e, () => {
            this.removeCssClass(el, 'animated')
            this.removeCssClass(el, cls)
            resolve()
          })
          this.addCssClass(el, 'animated')
          this.addCssClass(el, cls)
        })
      })
    },
    // 显示
    show() {
      this.$el.style.display = 'block'
      this.captureParentOverflow()
      this.forceParentOverflow()
      this.updateContainerSize()
      this.updateMainSize()
      this.anime(this.mainEl, 'fadeInDown')

      this.isHidden = false
    },
    // 隐藏
    hide() {
      if (this.isHidden) return
      this.anime(this.$el, 'fadeOut').then(() => {
        this.$el.style.display = 'none'
        this.resumeParentOverflow()
        this.$emit('input', false)
        this.isHidden = true
      })
    },
    // 点击蒙层消失
    maskClicked() {
      if (!this.clickMask2Close) return
      this.hide()
    }
  },
  mounted() {
    // 如果是全屏显示，则把内容显示到body下
    if (this.wholeWindow) {
      this.$el.parentNode.removeChild(this.$el)
      document.body.appendChild(this.$el)
    }
  }
}
</script>

<style>
.hsy-dialog {
  display: none;
  font: 10px "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", SimSun, sans-serif;
  position: absolute;
  top: 0;
  left: 0;
}

.hsy-dialog .mask {
  width: 100%;
  height: 100%;
}

.hsy-dialog .main {
  position: absolute;
  margin: 0 auto;
  top: 50%;
}

.hsy-dialog .main>div {
  margin: 0 auto;
}

.hsy-dialog .main .title {
  position: relative;
  color: #0E3569;
  height: 40px;
  line-height: 40px;
  overflow: hidden;
}

.hsy-dialog .main .title .content>* {
  width: 80%;
  min-width: 85px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.hsy-dialog .main .title .content {
  font-size: 1.8em;
  min-height: 20px;
  vertical-align: middle;
}

.hsy-dialog .main .title .btnClose {
  width: 20px;
  height: 20px;
  float: right;
  cursor: pointer;
  position: absolute;
  right: 0;
  top: 50%;
  transform: translate(0, -50%);
}

.hsy-dialog .main .body {
  max-width: 46vw;
  color: #6A7288;
  font-size: 1.2em;
  border-top: 1px solid #eee;
  margin-top: 5px;
  padding-top: 15px;
}

.hsy-dialog.animated,
.hsy-dialog .animated {
  animation-duration: 0.35s;
  animation-fill-mode: both;
}

@keyframes fadeInDown {
  from {
    opacity: 0;
    -webkit-transform: translate3d(0, -30%, 0);
    transform: translate3d(0, -30%, 0);
  }
  to {
    opacity: 1;
    -webkit-transform: none;
    transform: none;
  }
}

.hsy-dialog.fadeInDown,
.hsy-dialog .fadeInDown {
  animation-name: fadeInDown;
}

@keyframes fadeOut {
  from {
    opacity: 1;
  }
  to {
    opacity: 0;
  }
}

.hsy-dialog.fadeOut,
.hsy-dialog .fadeOut {
  animation-name: fadeOut;
}

.spinner {
  margin: 0 auto;
  width: 50px;
  height: 50px;
  position: relative;
}

.container1 > div, .container2 > div, .container3 > div {
  width: 15px;
  height: 15px;
  background-color: #333;

  border-radius: 100%;
  position: absolute;
  -webkit-animation: bouncedelay 1.2s infinite ease-in-out;
  animation: bouncedelay 1.2s infinite ease-in-out;
  -webkit-animation-fill-mode: both;
  animation-fill-mode: both;
}

.spinner .spinner-container {
  position: absolute;
  width: 100%;
  height: 100%;
}

.container2 {
  -webkit-transform: rotateZ(45deg);
  transform: rotateZ(45deg);
}

.container3 {
  -webkit-transform: rotateZ(90deg);
  transform: rotateZ(90deg);
}

.circle1 { top: 0; left: 0; }
.circle2 { top: 0; right: 0; }
.circle3 { right: 0; bottom: 0; }
.circle4 { left: 0; bottom: 0; }

.container2 .circle1 {
  -webkit-animation-delay: -1.1s;
  animation-delay: -1.1s;
}

.container3 .circle1 {
  -webkit-animation-delay: -1.0s;
  animation-delay: -1.0s;
}

.container1 .circle2 {
  -webkit-animation-delay: -0.9s;
  animation-delay: -0.9s;
}

.container2 .circle2 {
  -webkit-animation-delay: -0.8s;
  animation-delay: -0.8s;
}

.container3 .circle2 {
  -webkit-animation-delay: -0.7s;
  animation-delay: -0.7s;
}

.container1 .circle3 {
  -webkit-animation-delay: -0.6s;
  animation-delay: -0.6s;
}

.container2 .circle3 {
  -webkit-animation-delay: -0.5s;
  animation-delay: -0.5s;
}

.container3 .circle3 {
  -webkit-animation-delay: -0.4s;
  animation-delay: -0.4s;
}

.container1 .circle4 {
  -webkit-animation-delay: -0.3s;
  animation-delay: -0.3s;
}

.container2 .circle4 {
  -webkit-animation-delay: -0.2s;
  animation-delay: -0.2s;
}

.container3 .circle4 {
  -webkit-animation-delay: -0.1s;
  animation-delay: -0.1s;
}

@-webkit-keyframes bouncedelay {
  0%, 80%, 100% { -webkit-transform: scale(0.0) }
  40% { -webkit-transform: scale(1.0) }
}

@keyframes bouncedelay {
  0%, 80%, 100% {
    transform: scale(0.0);
    -webkit-transform: scale(0.0);
  } 40% {
    transform: scale(1.0);
    -webkit-transform: scale(1.0);
  }
}
</style>
