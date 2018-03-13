<template>
    <div class="header">
        <div class="imageBox" @click="back">
            <image :src="backUrl" resize="contain" class="back"></image>
            <text class="backText">返回</text>
        </div>
        <text class="text">{{head}}</text>
        <text class="text flexRight"></text>
    </div>
</template>

<script>
    import self from '../self.config';

    const navigator = weex.requireModule('navigator');

    export default {
        name: "title-bar",
        props:["head"],
        data(){
            return{
                backUrl:self.imgUrl+'btn_arrow_top.png',
            }
        },
        methods:{
            back:function () {
                navigator.pop();
            }
        }
    }
</script>

<style scoped>
    .header{
        width: 750px;
        padding-left: 30px;
        padding-right: 30px;
        background-color: #3c98e6;
        font-size: 34px;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        height: 88px;
    }
    .imageBox{
        flex-direction: row;
        flex: 4;
        align-items: center;
    }
    .back{
        width: 26px;
        height: 42px;
    }
    .backText{
        font-size: 28px;
        color: #fff;
        margin-left: 10px;
    }
    .text{
        color: #fff;
        font-size: 34px;
        text-align: center;
        flex: 10;
        lines: 1;
        text-overflow: ellipsis;
    }
    .flexRight{
        flex: 4;
        font-size: 28px;
    }
</style>