<template>
    <div class="wrapper">
        <div class="left">
            <text class="dataType textFont">{{title}}</text>
            <text class="numFont">{{total}}</text>
        </div>
        <div class="left">
            <text class="dataType textFont">现付</text>
            <text class="numFont">{{unreach.toFixed(2)}}</text>
        </div>
        <div class="left">
            <text class="dataType textFont">补缴</text>
            <text class="numFont">{{back_freight.toFixed(2)}}</text>
        </div>
        <div class="right">
            <text class="dataType textFont">运费合计</text>
            <text class="numFont colorFont">￥{{amount.toFixed(2)}}</text>
        </div>
    </div>
</template>

<script>
    export default {
        name: "data",
        props:["title","total","amount","back_freight","unreach"]
    }
</script>

<style scoped>
    .wrapper{
        flex-direction: row;
        align-items: center;
        width: 690px;
        height: 188px;
    }
    .dataType{
        margin-bottom: 24px;
    }
    .textFont{
        font-size: 28px;
        color: #666666;
        text-align: center;
    }
    .numFont{
        font-size: 30px;
        color: #333333;
        height: 40px;
        line-height: 40px;
        text-align: center;
    }
    .colorFont{
        color: #feae49;
    }
    .left{
        flex: 1;
    }
    .right{
        flex: 1;
    }
</style>