<template>
    <div class="wrapper" :class="[isBottom ? 'bottomOne':'bottomTwo']">
        <div class="mask" :style="{opacity:maskOpacity}" ref="mask" @click="toggle"></div>
        <div class="list" ref="list">
            <sort v-if="isSort" :sort="sort" @change="change" class="position"></sort>
        </div>
        <div class="title">
            <div class="titleBox">
                <div class="filtrate" @click="unfold(1)">
                    <text class="text">{{getTitle}}</text>
                    <image :src="arrowDown" class="image" resize="contain" ref="arrowTwo"></image>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import self from '../self.config';
    import sort from './sort.vue';

    const animation = weex.requireModule("animation");
    const modal = weex.requireModule("modal");
    const stream = weex.requireModule("stream");
    export default {
        name: "filtrate2",
        data(){
            return{
                arrowDown:self.imgUrl+'icon_arrow_down.png',
                isBottom:false,
                sort_id:'0',
                pause_value:"",
                isTitle:false,
                isStation:false,
                isSort:false,
                isUnfold:false,
                col:[],
                sort:[{
                    id:"0",
                    text:"金额排序",
                    sortStr:"",
                    isActive: true
                },{
                    id:"1",
                    text:"金额高到低",
                    sortStr:"desc",
                    isActive: false
                },{
                    id:"2",
                    text:"金额低到高",
                    sortStr:"asc",
                    isActive: false
                }],
                color:"#dddddd",
                is_opacity:0.1,
                is_translate: false,
                is_rotate:[false,false],
                maskOpacity:0,
                flag:true,
                index:0,
            }
        },
        computed:{
          getTitle:function () {
              const id = this.sort_id;
              return this.sort.filter(function (value) {
                  return value.id == id;
              })[0].text;
          }
        },
        methods:{
            unfold:function(index){
                const _self = this;
                if(_self.flag){
                    _self.flag=false;
                    if(!_self.isUnfold){
                        _self.isBottom = true;
                        _self.index = index;
                        _self.changeView(index);
                        _self.isMask();
                        _self.rotate(index);
                        _self.collapse(_self.$refs.list);
                        _self.isUnfold = true;
                    }else {
                        if(index != _self.index){
                            _self.index = index;
                            _self.rotate(index);
                            _self.changeView(index);
                            _self.flag = true;
                        }else{
                            _self.isBottom = false;
                            _self.isMask();
                            _self.rotate(index);
                            _self.collapse(_self.$refs.list);
                            _self.isUnfold = false;
                        }
                    }
                }
            },
            changeView:function (index) {
                const _self = this;
                if(index == 0){
                    _self.isSort = false;
                    setTimeout(function () {
                        _self.isStation = true;
                    },50);
                }else if(index == 1){
                    _self.isStation = false;
                    setTimeout(function () {
                        _self.isSort = true;
                    },50);
                }
            },
            animation:function (ref,styles,timingFunction,duration,callback) {
                animation.transition(ref,{
                    styles:styles,
                    duration: duration,
                    timingFunction:timingFunction
                },callback)
            },
            isMask:function () {
                this.maskOpacity = this.maskOpacity == 0 ? 0.3 : 0;
            },
            collapse:function (ref) {
                const _self = this;
                let translatey = _self.is_translate ? -292 : 292;
                _self.animation(ref,{
                    transform:"translateY("+translatey+"px)"
                }, "ease" ,200,function () {
                    _self.is_translate = !_self.is_translate;
                    _self.flag = true;
                })
            },
            rotate:function (index) {
                const _self = this;
                let ref;
                _self.is_rotate[index] = !_self.is_rotate[index];
                for(let i in _self.is_rotate){
                    if(i!=index){
                        _self.is_rotate[i] = false;
                    }
                    if(i==0){
                        ref = _self.$refs.arrowOne;
                    }else if(i==1){
                        ref = _self.$refs.arrowTwo;
                    }else if(i==2){
                        ref = _self.$refs.arrowThree;
                    }
                    let ratate = _self.is_rotate[i] ? 180 : 0;
                    _self.animation(ref,{
                        transform:"rotate("+ratate+"deg)"
                    },"linear",200)
                }
            },
            change:function (index,str) {
                for(let i in this.sort){
                    this.sort[i].isActive = false;
                }
                this.sort[index].isActive = true;
                this.sort_id = index;
                this.toggle();
                this.$emit("fresh",str);
            },
            toggle:function () {
                const _self = this;
                _self.unfold(_self.index);
            },
            cancle:function () {
                const _self = this;
                _self.unfold(_self.index);
            },
        },
        components:{
            "sort":sort,
        }
    }
</script>

<style scoped>
    .bottomOne{
        align-items: center;
        width: 750px;
        position: absolute;
        bottom: 0;
    }
    .bottomTwo{
        align-items: center;
        width: 750px;
        position: absolute;
    }
    .title{
        background-color: #fff;
        width: 750px;
        height: 80px;
        align-items: center;
        justify-content: center;
    }
    .titleBox{
        width: 690px;
        flex-direction: row;
        height: 80px;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
        /*position: absolute;*/
        background-color: #fff;
        /*top: 0;*/
    }
    .filtrate{
        flex-direction: row;
        flex: 1;
        justify-content: center;
        align-items: center;
    }
    .text{
        font-size: 28px;
        color: #666666;
        text-overflow: ellipsis;
        lines:1;
    }
    .image{
        width: 18px;
        height: 18px;
        margin-left: 12px;
    }
    .list{
        align-items: center;
        background-color: #fff;
        width: 750px;
        position: absolute;
        top: -214px;
    }
    .mask{
        position: absolute;
        bottom: 0;
        top:80px;
        left: 0;
        right: 0;
        background-color: black;
    }
</style>