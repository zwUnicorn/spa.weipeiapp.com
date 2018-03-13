<template>
    <div class="wrapper row" bubble="true">
        <div class="left" @click="dothis">
            <div class="result">
                <text class="sum">￥{{result.sum}}</text>
                <div class="row">
                    <text class="font">运费&nbsp;￥{{result.freight}}</text>
                    <text class="font">货款&nbsp;￥{{result.payment}}</text>
                </div>
            </div>
        </div>
        <div class="right row" @click.stop="restore">
            <text class="color">恢复交易</text>
            <text class="color num">（{{result.num}}单）</text>
        </div>
    </div>
</template>

<script>
    import self from '../self.config';
    var stream = weex.requireModule('stream');
    const modal = weex.requireModule('modal');

    export default {
        name: "result",
        props:["result"],
        methods: {
            restore: function () {
                const _self = this;
                modal.confirm({
                    message:"确认恢复交易",
                    okTitle:"确认",
                    cancelTitle:"取消",
                },result => {
                   if(result=="确认"){
                       _self.$emit('restoreRefresh')
                   }
                });
            }
        }
    }
</script>

<style scoped>
    .wrapper{
        width: 750px;
        height: 98px;
        border-top-color: #dddddd;
        border-top-width: 1px;
        background-color: #fff;
    }
    .row{
        flex-direction: row;
    }
    .sum{
        font-size: 28px;
        color: #3c98e6;
    }
    .font{
        font-size: 22px;
        color: #999999;
        margin-right: 24px;
    }
    .left{
        width: 430px;
        justify-content: center;
        background-color: #fff;
    }
    .result{
        margin-left: 30px;
    }
    .right{
        width: 320px;
        align-items: center;
        justify-content: center;
        background-color: #3c98e6;
    }
    .color{
        color: #fff;
        font-size: 30px;
    }
    .num{
        font-size: 22px;
    }
</style>