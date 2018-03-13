<template>
    <div class="wrapper">
        <div class="outBox">
            <div class="name box">
                <div class="namebox box">
                    <text class="deliverymanName">{{deliveryman.goodses_no}}</text>
                    <text v-if="deliveryman.is_reached_freight_cashbacked && deliveryman.is_goods_expense_cashbacked" class="sheetstatus">回</text>
                </div>
                <text class="fontTotal">￥{{(deliveryman.reached_receivable_freight+deliveryman.receivable_goods_expense).toFixed(2)}}</text>
            </div>
            <div class="numBox box">
                <div class="numInfo box">
                    <text class="fontOne">运费￥{{deliveryman.reached_receivable_freight+""+(deliveryman.is_reached_freight_cashbacked?"（已回款）":"")}}</text>
                    <text class="fontOne">货款￥{{deliveryman.receivable_goods_expense+""+(deliveryman.is_goods_expense_cashbacked?"（已回款）":"")}}</text>
                </div>
            </div>
            <div class="info box">
                <text class="font">{{deliveryman.sender}} - {{deliveryman.receiver}}</text>
            </div>
        </div>
    </div>
</template>

<script>
    import self from "../self.config"

    export default {
        name: "deliverysheet",
        props:["deliveryman"],
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
        flex: 1;
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
        align-items: center;
        justify-content: space-between;
    }
    .deliverymanName{
        font-size: 32px;
        color: #333333;
        padding-right: 10px;
    }
    .numBox{
        justify-content: space-between;
        align-items: center;
        margin-bottom: 8px;
    }
    .namebox{
        align-items: center;
    }
    .sheetstatus{
        width: 36px;
        height: 36px;
        text-align: center;
        font-size: 22px;
        line-height: 36px;
        border-radius: 4px;
        background-color: #3c98e6;
        color: #fff;
    }
</style>