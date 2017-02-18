<template>
<div class="hsy-dialog">
  <div class="mask" :style="{backgroundColor: maskColor}"></div>
  <div class="main">
    <div class="inner">
      <div class="title">
        <div class="content">
          <slot name="title"></slot>
        </div>
        <div class="btnClose" @click="hide"></div>
      </div>
      <div class="body">
        <slot name="body"></slot>
      </div>
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
      msg: 'hello vue-component!',
      width: '',
      height: '',
      bodyOverflowX: '',
      bodyOverflowY: '',
      isHidden: true
    }
  },
  props: {
    maskColor: {
      type: String,
      default: 'rgba(0, 0, 0, 0.3)'
    },
    closeButton: {
      type: Boolean,
      default: true
    },
    clickMask2Close: {
      type: Boolean,
      default: true
    },
    value: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    mainEl() {
      return this.$el.querySelector('.main')
    }
  },
  watch: {
    value(val) {
      if (val) {
        this.show()
      } else {
        this.hide()
      }
    }
  },
  methods: {
    rect(el) {
      let rect = el.getBoundingClientRect()
      if (rect.width !== 0 || rect.height !== 0) {
        return rect
      }

      let style = window.getComputedStyle(el)
      let display = style.getPropertyValue('display')
      let top = style.getPropertyValue('top')
      let left = style.getPropertyValue('left')

      el.style.top = '-1000px'
      el.style.left = '-1000px'
      el.style.display = 'inline-block'
      rect = el.getBoundingClientRect()

      el.style.display = display
      el.style.top = top
      el.style.left = left

      return rect
    },
    updateContainerSize() {
      this.$el.style.width = document.documentElement.clientWidth + 'px'
      this.$el.style.height = document.documentElement.clientHeight + 'px'
    },
    updateMainSize() {
      let mainEl = this.$el.querySelector('.main')
      let mainElRect = this.rect(mainEl)
      mainEl.style.width = mainElRect.width + 'px'
      mainEl.style.left = '0px'
      mainEl.style.right = '0px'
    },
    captureBodyOverflow() {
      let style = window.getComputedStyle(document.body)
      this.bodyOverflowX = style.getPropertyValue('overflowX')
      this.bodyOverflowY = style.getPropertyValue('overflowY')
    },
    forceBodyOverflow() {
      document.body.style.overflow = 'hidden'
    },
    resumeBodyOverflow() {
      document.body.style.overflowX = this.bodyOverflowX
      document.body.style.overflowY = this.bodyOverflowY
    },
    addCssClass(el, cls) {
      let clsList = el.className.split(' ')
      if (clsList.indexOf(cls) === -1) {
        clsList.push(cls)
        el.className = clsList.join(' ')
      }
    },
    removeCssClass(el, cls) {
      let clsList = el.className.split(' ')
      if (clsList.indexOf(cls) !== -1) {
        el.className = clsList.filter((c) => {
          return c !== cls
        }).join(' ')
      }
    },
    anime(el, cls) {
      return new Promise((resolve) => {
        ANIME_EVENTS.forEach((e) => {
          el.addEventListener(e, () => {
            console.log(1, cls)
            this.removeCssClass(el, 'animated')
            this.removeCssClass(el, cls)
            resolve()
          })
          this.addCssClass(el, 'animated')
          this.addCssClass(el, cls)
        })
      })
    },
    show() {
      this.$el.style.display = 'block'
      this.captureBodyOverflow()
      this.forceBodyOverflow()
      this.updateContainerSize()
      this.updateMainSize()
      this.anime(this.mainEl, 'fadeInDown')

      this.isHidden = false
    },
    hide() {
      if (this.isHidden) return
      this.anime(this.$el, 'fadeOut').then(() => {
        this.$el.style.display = 'none'
        this.$emit('input', false)
        this.isHidden = true
      })
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
  top: 20%;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
  background: #fff;
  border-radius: 3px;
  font-size: 1.2em;
}

.hsy-dialog .main>div {
  margin: 0 auto;
  padding: 20px;
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
  background: url('assets/images/close.svg') no-repeat center center;
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
</style>