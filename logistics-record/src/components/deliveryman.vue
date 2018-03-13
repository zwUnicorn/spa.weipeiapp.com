<template>
    <div class="wrapper">
        <div class="outBox">
            <div class="name box">
                <text class="deliverymanName">{{deliveryman.realname}}</text>
                <text class="font leftDote">共{{deliveryman.total_sheet}}单</text>
            </div>
            <div class="numBox box">
                <div class="numInfo box">
                    <text class="fontOne">运费￥{{deliveryman.freight_amount}}</text>
                    <text class="fontOne">货款￥{{deliveryman.goods_expense_amount}}</text>
                </div>
                <text class="fontTotal">￥{{(deliveryman.freight_amount+deliveryman.goods_expense_amount).toFixed(2)}}</text>
            </div>
            <div class="info box">
                <text class="font">已回运费￥{{deliveryman.freight_cashbacked_amount}}</text>
                <text class="font">已回货款￥{{deliveryman.goods_expense_cashbacked_amount}}</text>
                <text class="font leftDote">合计￥{{(deliveryman.freight_cashbacked_amount+deliveryman.goods_expense_cashbacked_amount).toFixed(2)}}</text>
            </div>
        </div>
    </div>
</template>

<script>
    import self from "../self.config"
    const navigator = weex.requireModule("navigator");

    export default {
        name: "deliveryman",
        props:["deliveryman"],
        methods:{
            redirectList:function (id) {
                navigator.push({
                    url: self.hostUrl + 'info.js?id=' + id
                })
            },
        }
    }
</script>

<style scoped>
    .wrapper{
        width: 750px;
        align-items: flex-end;
    }
    .font{
        color: #999999;
        font-size: 24px;
        margin-right: 24px;
    }
    .fontOne{
        font-size: 28px;
        color: #666666;
        margin-right: 24px;
    }
    .fontTotal{
        font-size: 30px;
        color: #feae49;
    }
    .outBox{
        width: 720px;
        padding-top: 30px;
        padding-bottom: 30px;
        padding-right: 30px;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
    }
    .box{
        flex-direction: row;
        align-items: center;
    }
    .name{
        margin-bottom: 5px;
    }
    .deliverymanName{
        font-size: 32px;
        color: #333333;
        padding-right: 24px;
    }
    .numBox{
        justify-content: space-between;
        align-items: center;
        margin-bottom: 8px;
    }
    .leftDote{
        padding-left: 24px;
        border-left-color: #dddddd;
        border-left-width: 1px;
    }
</style>