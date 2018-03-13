<template>
    <div class="wrapper">
        <div class="left">
            <text class="dataType textFont">{{merchant}}</text>
            <text class="numFont">{{datas.total_merchants}}</text>
        </div>
        <div class="cutLine"></div>
        <div class="center">
            <div class="centerLeft">
                <text class="dataType textFont">到付</text>
                <text class="numFont">￥{{datas.total_reached_receivable_freight}}</text>
            </div>
            <div class="centerRight">
                <text class="textFont dataType">货款</text>
                <text class="numFont">￥{{datas.total_receivable_goods_expense}}</text>
            </div>
        </div>
        <div class="cutLine"></div>
        <div class="right">
            <text class="dataType textFont">合计</text>
            <text class="numFont colorFont">￥{{datas.amount}}</text>
        </div>
    </div>
</template>

<script>
    export default {
        name: "data",
        props:["datas","merchant"]
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
    .cutLine{
        width: 3px;
        height: 80px;
        background-color: #dddddd;
    }
    .left{
        flex: 2;
    }
    .center{
        flex-direction: row;
        flex: 6;
    }
    .centerLeft{
        flex: 2;
    }
    .centerRight{
        flex: 3;
    }
    .right{
        flex: 3;
    }
</style>