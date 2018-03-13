<template>
    <div class="wrapper">
        <div class="outBox" v-for="items in merchant" @click="redirectList(items.id)">
            <div class="name box">
                <text class="merchantName">{{items.name}}</text>
                <text class="merchantSum">￥{{items.meta.amount}}</text>
            </div>
            <div class="info box">
                <text class="font">运费&nbsp;￥{{items.meta.total_reached_receivable_freight}}</text>
                <text class="font">货款&nbsp;￥{{items.meta.total_receivable_goods_expense}}</text>
                <text class="font num">共{{items.meta.total_paused_sheet}}单</text>
            </div>
        </div>
    </div>
</template>

<script>
    import self from "../self.config"
    const navigator = weex.requireModule("navigator");

    export default {
        name: "merchant",
        props:["merchant"],
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
        align-items: center;
    }
    .font{
        color: #999999;
        font-size: 26px;
        margin-right: 24px;
    }
    .outBox{
        width: 690px;
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
        justify-content: space-between;
        margin-bottom: 5px;
    }
    .merchantName{
        font-size: 30px;
        color: #333333;
        lines: 1;
        text-overflow: ellipsis;
    }
    .merchantSum{
        font-size: 32px;
        color: #feae49;
    }
    .num{
        padding-left: 24px;
        border-left-color: #dddddd;
        border-left-width: 1px;
    }
</style>