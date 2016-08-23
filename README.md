# vue-scrollBar

## 引入组件
`import itemSwipe from '../../components/itemSwipe.vue'`

##组件的定义
```
<template>
    <div class="cp-swiper" @touchmove.prevent="touchmove($event)"  @touchstart="touchstart($event)" @touchend="touchend" >
        <div class="cp-swiper-wrap" :class="{'snipe': snipeIng}" :style="{transform:'translateX('+translateX+'px)'}">
            <ul v-el:swiper class="swiper" :style="{width: wrapWidth + 'px'}">
                <slot name="last"></slot>
                <slot name="first"></slot>
                <slot name="middle"></slot>
                <slot name="last"></slot>
                <slot name="first"></slot>
            </ul>
        </div>
        <ul class="swiper-dot">
            <li :class="{'cur': Math.abs(curIndex)==i}" v-for="i in itemLen"></li>
        </ul>
    </div>
</template>
```
##组件事件的定义
```
<script type="text/ecmascript-6">
import {toast} from '../modules/util'
export default {

  //==数据配置===
    data() {
      return {
        state: 0,
        startX: 0,
        lastTranslateX: 0,
        translateX: 0,
        snipeIng: false,
        touching: false,
        snipeWidth: 0,
        curIndex: 0,
        itemLen: 0,
        autoTime: 5000
      }
    },
    computed: {
        maxWidth () {
            return - this.snipeWidth *    (this.itemLen - 1);
        },
        wrapWidth () {
            return this.snipeWidth *    (this.itemLen + 2);
        }
    },
    props: {
        // isShow: {
        //     default: false,
        //     type: Boolean,
        //     twoWay: true  
        // },
    },
    ready () {
        this.snipeWidth = this.$el.clientWidth;
        //减去两个头尾重复的
        this.itemLen = this.$els.swiper.getElementsByClassName('swipe-item').length - 2;
        
        var that = this;
        this.timer = setTimeout(function(){
            that.swipe(that.curIndex - 1);
        }, this.autoTime);

        window.addEventListener('resize', this.resize);
    },
    destroyed() {
        clearTimeout(this.timer);
        window.removeEventListener('resize', this.resize);
    },
    methods: {
        touchstart (event) {
            if (this.snipeIng) return;
            this.startY = event.touches[0].pageY;
            this.startX = event.touches[0].pageX;
            this.lastTranslateX = this.translateX;
        }, 
        touchmove (event) {
            var that = this;
            clearTimeout(this.timer);
            if (that.snipeIng) return;
            that.translateX = that.lastTranslateX + event.touches[0].pageX - that.startX;
            
        },
        touchend (event) {
            if (this.snipeIng) return;

            var step = 0, plus, mo = this.translateX % this.snipeWidth;
            var direction = this.translateX > this.lastTranslateX;
            if (direction && this.translateX < 0) {    //往右滑
                plus = Math.abs(mo) > this.snipeWidth * 3 / 4 ? 1 : 0; 
            } else {
                plus = Math.abs(mo) > this.snipeWidth / 4 ? 1 : 0; 
            }

            if (this.translateX >= 0) {
                step = Math.floor(this.translateX / this.snipeWidth) + plus;
            } else {
                step = Math.ceil(this.translateX / this.snipeWidth) - plus;
            }
            this.swipe(step);
        },
        swipe (x) {
            var translateX = x * this.snipeWidth;
            if (translateX > 0) {
                this.translateX = this.maxWidth - this.snipeWidth + this.translateX;
                translateX = this.maxWidth;
            } else if (translateX < this.maxWidth) {
                this.translateX = this.snipeWidth - this.maxWidth + this.translateX;
                translateX = 0;   
            }
            this.curIndex = translateX / this.snipeWidth;
            var that = this;
            this.$nextTick(function () {
                setTimeout(function(){
                    that.snipeIng = true;
                    that.translateX = translateX;
                    
                    setTimeout(function(){
                        that.snipeIng = false;
                        clearTimeout(that.timer);
                        that.timer = setTimeout(function(){
                            that.swipe(that.curIndex - 1);
                        }, that.autoTime);
                    }, 400);
                }, 20);
            });
        },
        resize () {
            this.snipeWidth = this.$el.clientWidth;
        }
    }
}
</script>
```
## 组件的使用
```
<item-swipe class="faq-rank">
          <li slot="first" class="swipe-item" v-link="{'path': '/rank'}">
                <img class="c-img" src="/assets/images/sale-rank.jpg" >
          </li>
          <li slot="last" class="swipe-item" v-link="{'path': '/faq'}">
               <img class="c-img" src="/assets/images/faq-banner.png" >
          </li>
</item-swipe>
```
