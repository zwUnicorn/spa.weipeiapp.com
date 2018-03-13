<template>
    <div class="barbox">
        <div class="box firstbox">
            <text class="price">￥{{info.amount_to_fen}}</text>
            <text class="state">{{info.status}}</text>
        </div>
        <div class="box">
            <text class="content">{{info.title}}</text>
            <text class="date">{{info.created_at.substr(0,10)}}</text>
        </div>
    </div>
</template>
<style scoped>
    .barbox{
        width: 718px;
        margin-left: 32px;
        border-top-color: #dddddd;
        border-top-width: 1px;
        padding-bottom: 24px;
        padding-top: 24px;
        padding-right: 32px;
    }
    .box{
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
    }
    .firstbox{
        margin-bottom: 8px;
    }
    .price{
        font-size: 38px;
        color: #f5a623;
        position: relative;
        right: 3px;
    }
    .state{
        color: #44b3e1;
        font-size: 28px;
    }
    .content{
        font-size: 26px;
        color: #666666;
        flex: 3;
        lines:1;
        text-overflow: ellipsis;
    }
    .date{
        font-size: 24px;
        color: #999999;
        flex: 1;
        margin-left: 64px;
        text-align: right;
    }
</style>
<script>
    export default {
        props:['info'],
    }
</script>