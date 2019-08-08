<template>
    <div class="slider" ref="slider">
        <div class="slider-group" ref="sliderGroup">
            <slot></slot>
        </div>
        <div class="dots">
            <span class="dot" :class="{active: currentPageIndex===index}" v-for="(item,index) in dots" :key="index"></span>
        </div>
    </div>
</template>

<script>
import {addClass} from 'common/js/dom'
import BScroll from 'better-scroll'

export default {
    name:'slider',
    props:{
        loop:{//是否可以循环轮播
            type:Boolean,
            default:true
        },
        autoPlay:{//自动轮播
            type:Boolean,
            default:true
        },
        interval:{//自动轮播间隔
            type:Number,
            default:4000
        }
    },
    data(){
        return{
            dots:[],
            currentPageIndex:0
        }
    },
    mounted(){
        setTimeout(()=>{
            this._setSliderWidth()
            this._initSlider()
            this._initDots()
            
            if(this.autoPlay){
                this._play()
            }
        },20)

        window.addEventListener('resize',()=>{
            if(!this.slider || !this.slider.enable){
                return
            }
            clearTimeout(this.resizeTimer)
            this.resizeTimer=setTimeout(()=>{
                if(this.slider.isIntransition){
                    this._onScrollEnd()
                }else{
                    if(this.autoPlay){
                        this._play()
                    }
                }
                this.refresh()
            },60)
        })
    },
    activated(){
        this.slider.enable()
        let pageIndex=this.slider.getCurrentPage().pageX
        this.slider.goToPage(pageindex,0,0)
        this.currentPageIndex=pageindex
        if(this.autoPlay){
            this._play()
        }
    },
    deactivated(){
        this.slider.dsidable()
        clearTimeout(this.timer)
    },
    beforeDestroy(){
        this.slider.disable()
        clearTimeout(this.timer)
    },
    methods:{
        refresh(){
            if(this.slider){
                this._setSliderWidth(true)
                this.slider.refresh()
            }
        },
        _setSliderWidth(isResize){
            this.children=this.$refs.sliderGroup.children

            let width=0
            let sliderWidth=this.$refs.slider.clientWidth
            for(let i=0;i<this.children.length;i++){
                let child=this.children[i]
                addClass(child,'slider-item')

                child.style.width = sliderWidth +'px'
                width += sliderWidth
            }
            if(this.loop){
                width += 2 * sliderWidth
            }
            this.$refs.sliderGroup.style.width = width + 'px'
        },
        _initSlider(){
            this.slider = new BScroll(this.$refs.slider,{
                scrollX:true,
                scrollY:false,
                momentum:false,
                snap:{
                    loop:this.loop,
                    threshold:0.3,
                    speed:400
                }
            })

            this.slider.on('scrollEnd',()=>{
                let pageIndex = this.slider.getCurrentPage().pageX
                this.currentPageIndex=pageIndex

                if(this.autoPlay){
                    this._play()
                }
            })

            this.slider.on('beforeScrollStart',()=>{
                if(this.autoPlay){
                    clearTimeout(this.timer)
                }
            })
        },
        _onscrollEnd(){
            let pageIndex=this.slider.getCurrentPage().pageX
            this.currentPageIndex=pageIndex
            if(this.autoPlay){
                this._play()
            }
        },
        _initDots(){
            this.dots=new Array(this.children.length-2)
        },
        _play(){
            let pageIndex=this.currentPageIndex + 1
            this.timer=setTimeout(()=>{
                this.slider.next()
            },this.interval)
        },
    }
}
</script>

<style lang="stylus">
@import "~common/stylus/variable"

.slider
    min-height 1px
    position relative
    .slider-group
        
        overflow hidden
        white-space nowrap
        .slider-item
            float left
            box-sizing border-box
            overflow hidden
            text-align center
            a
                display block
                width 100%
                overflow hidden
                text-decoration none
            img
                display block
                width 100%
    .dots
        position absolute
        right 0
        left 0
        bottom 12px
        transform translateZ(1px)
        text-align center
        font-size 0
        .dot
            display inline-block
            margin 0 4px
            width 8px
            height 8px
            border-radius 50%
            background $color-text-l
            &.active
                width 20px
                border-radius 5px
                background $color-text-l
</style>
