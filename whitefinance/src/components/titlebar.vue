<template>
    <div class="header">
        <div class="imageBox">
            <image :src="backUrl" resize="contain" class="back" @click="back"></image>
            <image class="close" :src="closeUrl" resize="contain" @click="close"></image>
        </div>
        <text class="text">{{head}}</text>
        <text class="text flexRight" @click="jump">{{subhead}}</text>
    </div>
</template>
<style scoped>
    .header{
        width: 750px;
        height: 128px;
        padding-left: 16px;
        padding-right: 16px;
        padding-top: 44px;
        background-color: #44b3e1;
        font-size: 34px;
        flex-direction: row;
        justify-content: center;
        align-items: center;
    }
    .imageBox{
        flex-direction: row;
        flex: 4;
        position: relative;
        left: 24px;
    }
    .back{
        width: 44px;
        height: 44px;
    }
    .close{
        width: 42px;
        height: 42px;
        margin-left: 28px;
    }
    .text{
        color: #fff;
        font-size: 34px;
        text-align: center;
        flex: 10;
    }
    .flexRight{
        flex: 4;
        font-size: 28px;
    }
</style>
<script>
    import self from '../self.config'
    const navigator = weex.requireModule('navigator')
    export default {
        props:["head","subhead"],
        data(){
            return{
                backUrl:self.imgUrl+'btn_back.png',
                closeUrl:self.imgUrl+'icon_close.png'
            }
        },
        methods:{
            back(){
                const url = weex.config.bundleUrl;
                if(url.indexOf("personal") >= 0){
                    weex.requireModule("nativeModule").redirectHome();
                }else{
                    navigator.pop()
                }
            },
            jump(){
                navigator.push({
                    url: self.hostUrl+'config.js'
                })
            },
            close:function () {
                weex.requireModule("nativeModule").redirectHome();
            }
        }
    }
</script>