<template>
    <div class="wrapper">
        <div class="dataBox" style="border-bottom-color: #dddddd;border-bottom-width: 1px">
            <div class="left">
                <text class="dataType textFont">运费总额</text>
                <text class="numFont">￥{{freight_amount.toFixed(2)}}</text>
            </div>
            <div class="left">
                <text class="dataType textFont">货款总额</text>
                <text class="numFont">￥{{goods_expense_amount.toFixed(2)}}</text>
            </div>
            <div class="right">
                <text class="dataType textFont">费用合计</text>
                <text class="numFont colorFont">￥{{(freight_amount+goods_expense_amount).toFixed(2)}}</text>
            </div>
        </div>
        <div class="dataBox">
            <div class="left">
                <text class="dataType textFont">已回款运费</text>
                <text class="numFont">￥{{freight_cashbacked_amount.toFixed(2)}}</text>
            </div>
            <div class="left">
                <text class="dataType textFont">已回款货款</text>
                <text class="numFont">￥{{goods_expense_cashbacked_amount.toFixed(2)}}</text>
            </div>
            <div class="right">
                <text class="dataType textFont">已回款合计</text>
                <text class="numFont colorFont">￥{{(freight_cashbacked_amount+goods_expense_cashbacked_amount).toFixed(2)}}</text>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "data",
        props:["freight_amount","goods_expense_amount","freight_cashbacked_amount","goods_expense_cashbacked_amount"]
    }
</script>

<style scoped>
    .wrapper{
        align-items: center;
    }
    .dataBox{
        align-items: center;
        flex-direction: row;
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
        border-left-color: #cccccc;
        border-left-width: 1px;
    }
</style>