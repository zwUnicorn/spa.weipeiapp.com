<template>
    <div class="wrapper" :class="[isBottom ? 'bottomOne':'bottomTwo']">
        <div class="mask" :style="{opacity:maskOpacity}" @click="toggle" ref="mask"></div>
        <div class="list" ref="list">
            <sort v-if="isShow" :sort="sort" @change="change"></sort>
        </div>
        <div class="title">
            <div class="titleBox">
                <div class="filtrate" @click="showChoiceTwo()">
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
    import pause from "./pause.vue";

    const animation = weex.requireModule("animation");
    const modal = weex.requireModule("modal");
    const dom = weex.requireModule("dom");
    const storage=weex.requireModule("storage");
    const stream = weex.requireModule("stream");
    const path = weex.config.bundleUrl;

    export default {
        name: "filtrate",
        data(){
            return{
                arrowDown:self.imgUrl+'icon_arrow_down.png',
                isBottom:false,
                sort_id:'0',
                pause_value:" ",
                isTitle:false,
                isShow:true,
                sort:[{
                    id:"0",
                    text:"金额大小排序",
                    sortStr:" ",
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
                is_opacity:0.1,
                is_translate: false,
                is_rotate:false,
                maskOpacity:0,
                flag:true,
                type:"",
                arrowsActive:false,
                unfoldOne:false,
                unfoldTwo:false
            }
        },
        created:function () {
            const _self = this;
            weex.requireModule("nativeModule").getToken(function callback(res) {
                stream.fetch({
                    method:"get",
                    url:self.devUrl+"api/company/delivery_sheet_attributes?type=pause_category",
                    type:"json",
                    headers:{
                        "Accept": "application/vnd.logistic.v3+json",
                        "Authorization": "bearer "+ res.token
                    }
                },res=>{
                    const data = res.data.data;
                    for (let i in data){
                        let sort = i+2;
                        _self.pause.push({
                            sort_id:sort,
                            value: data[i].value,
                            id:data[i].id,
                            isActive:false
                        })
                    }
                    storage.getItem("pauseValue",function (e) {
                        if(e.result=="failed"){

                        }else{
                            _self.isTitle = true;
                            _self.pause_value = e.data;
                            for(let i in _self.pause){
                                if(_self.pause[i].value == _self.pause_value){
                                    _self.pause[i].isActive = true;
                                }
                            }
                        }

                    })
                });
            });
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
            rotate:function (ref) {
                const _self = this;
                let ratate = _self.is_rotate ? 0 : 180;
                _self.animation(ref,{
                    transform:"rotate("+ratate+"deg)"
                },"linear",200,function () {
                    _self.is_rotate = !_self.is_rotate;
                })
            },
            pulldown:function (ref) {
                const _self =this;
                if(_self.flag){
                    _self.flag = !_self.flag;
                    this.isBottom = !this.isBottom;
                    this.isMask();
                    this.collapse(_self.$refs.list);
                    this.rotate(ref);
                }
            },
            showChoiceTwo:function () {
                const _self = this;
                _self.isShow = true;
                _self.type = "1";
                if(_self.unfoldOne){
                    _self.isShow = true;
                    _self.unfoldOne = false;
                    _self.unfoldTwo = true;
                    _self.rotate(_self.$refs.arrowOne);
                    _self.is_rotate = !_self.is_rotate;
                    _self.rotate(_self.$refs.arrowTwo);
                    _self.is_rotate = !_self.is_rotate;
                }else{
                    this.pulldown(_self.$refs.arrowTwo);
                    _self.unfoldTwo = !_self.unfoldTwo;
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
                if(_self.type=="0"){
                    this.showChoiceOne()
                }else {
                    this.showChoiceTwo()
                }
            },
            cancle:function () {
                this.showChoiceOne();
            },
            getId:function (ids,value) {
                this.isTitle = true;
                this.pause_value = value;
                this.$emit("getID",ids);
                this.showChoiceOne();
            }
        },
        components:{
            "sort":sort,
            "pause":pause
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
    }
    .titleBox{
        width: 690px;
        margin-left: 30px;
        margin-right: 30px;
        flex-direction: row;
        height: 80px;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
        position: absolute;
        background-color: #fff;
        top: 0;
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