<template>
  <div class="slider" ref="slider">
    <div class="slider-group" ref="sliderGroup">
      <slot></slot>
    </div>
    <div class="dots">
      <span class="dot" 
        v-for="(item, index) in dots" 
        :key="index" 
        :class="{active: currentPageIndex === index}">
      </span>
    </div>
  </div>
</template>


<script>
// 针对移动端
import BScroll from "better-scroll";
import { addClass } from "../../common/js/dom";

export default {
  props: {
    loop: {
      type: Boolean,
      default: true
    },
    autoPlay: {
      type: Boolean,
      default: false
    },
    interval: {
      type: Number,
      default: 400
    }
  },
  name: "Slider",
  data: () => {
    return {
      dots: [],
      currentPageIndex: 0
    };
  },
  mounted() {
    setTimeout(() => {
      this._setSliderWidth(),
      this._initDots()
      this._initSlider()

      if (this.autoPlay) {
        this._play()
      }
    }, 200)

    window.addEventListener('resize', () => {
      if (!this.slider) {
          return
      }
      this._setSliderWidth(true)
      this.slider.refresh()// 刷新
    })
  },
  methods: {
    _setSliderWidth(isResize) {
      this.children = this.$refs.sliderGroup.children
      //console.log(this.children.length)
      let width = 0;
      let sliderWidth = this.$refs.slider.clientWidth; // slider dom节点
      for (let i = 0; i < this.children.length; i++) {
        let child = this.children[i];
        // console.log(this.children.length) == bug
        // 动态添加class
        addClass(child, "slider-item");

        child.style.width = sliderWidth + "px";
        width += sliderWidth;
        // console.log(width);
      }

      if (this.loop && isResize) {
        width += 2 * sliderWidth;
      }
      // 为轮播外部的元素添加宽度
      this.$refs.sliderGroup.style.width = width + "px";
    },
    // 初始化slider
    _initSlider() {
      this.slider = new BScroll(this.$refs.slider, {
        scrollX: true,
        scrollY: false,
        momentum: false,
        snap: true,
        snapLoop: this.loop,
        snapThreshold: 0.3,
        snapSpeed: 400,
        //click: true,
      })
      // dots
      this.slider.on('scrollEnd', () => {
        let pageIndex = this.slider.getCurrentPage().pageX
        if (this.loop) {
          pageIndex -=1
          this.currentPageIndex = pageIndex

          //  清除定时器
          if (this.autoPlay) {
            clearTimeout(this.timer)
            this._play()
          }
        }
      })
    },
    _initDots() {
      this.dots = new Array(this.children.length)
    },
    _play() {
      let pageIndex = this.currentPageIndex + 1
      if (this.loop) {
        pageIndex += 1
      }
      this.timer = setTimeout(() => {
        this.slider.goToPage(pageIndex, 0, 400)
      }, this.interval)
    }
  },
  destroyed() {
    clearTimeout(this.timer)
  }
};
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import '../../common/stylus/variable';

.slider {
  min-height: 1px;

  .slider-group {
    position: relative;
    overflow: hidden;
    white-space: nowrap;

    .slider-item {
      float: left;
      box-sizing: border-box;
      overflow: hidden;
      text-align: center;

      a {
        display: block;
        width: 100%;
        overflow: hidden;
        text-decoration: none;
      }

      img {
        display: block;
        width: 100%;
      }
    }
  }

  .dots {
    position: absolute;
    right: 0;
    left: 0;
    bottom: 12px;
    text-align: center;
    font-size: 0;

    .dot {
      display: inline-block;
      margin: 0 4px;
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: $color-text-l;

      &.active {
        width: 20px;
        border-radius: 5px;
        background: $color-text-ll;
      }
    }
  }
}
</style>
