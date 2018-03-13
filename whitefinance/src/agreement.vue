<template>
    <div class="wrapper">
        <title-bar :head="title" :subhead="subtitle"></title-bar>
        <scroller class="scroller">
            <text class="scrollertext">{{text}}</text>
        </scroller>
        <div class="btn-group">
            <text class="text no" @click="pop">不同意</text>
            <text class="text yes" @click="jump">同意</text>
        </div>
    </div>
</template>

<style scoped>
    .wrapper {
        align-items: center;
    }
    .scroller{
        width: 750px;
        position:fixed;
        top: 128px;
        bottom: 130px;
    }
    .scrollertext{
        width: 750px;
        padding-left: 32px;
        padding-right: 32px;
        padding-top: 34px;
        padding-bottom: 10px;
        color: #666666;
        font-size: 28px;
        text-align: justify;
        text-indent: 2em;
    }
    .btn-group{
        width: 750px;
        height: 120px;
        position: fixed;
        flex-direction: row;
        bottom: 0;
        justify-content: center;
        align-items: center;
        border-top-width: 2px;
        border-top-color: #dddddd;
        background-color: #fff;
    }
    .text{
        width: 300px;
        margin-left: 16px;
        margin-right: 16px;
        height: 80px;
        text-align: center;
        line-height: 80px;
        font-size: 28px;
        border-radius: 10px;
    }
    .yes{
        background-color: #44b3e1;
        color: #fff;
    }
    .no{
        border-width: 2px;
        border-color: #dddddd;
        color: #666666;
    }
</style>

<script>
    import self from './self.config'
    import titlebar from './components/titlebar.vue'

    const navigator = weex.requireModule('navigator')
    export default {
        data () {
            return{
                title: '维配白条使用协议',
                subtitle: '',
                text: '48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退' +
                '款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时' +
                '快速退款48小时快速退款48小时快速退款48小时快速退款48小时快时快速退款48小时快速退款48小时快速退款48小时快时快速退款48小时快速退款48小时快速退款48小时快时快速退款48小时快速退款48小时快速退款48小时快' +
                '时快速退款48小时快速退款48小时快速退款48小时快时快速退款48小时快速退款48小时快速退款48小时快时快速退款48小时快速退款48小时快速退款48小时快' +
                '时快速退款48小时快速退款48小时快速退款48小时快时快速退款48小时快速退款48小时快速退款48小时快' +
                '速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款4退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退' +
                '退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退' +
                '退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退' +
                '退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退' +
                '退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退退款48小时快速退款48小时快速退8' +
                '小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退' +
                '款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时快速退款48小时' +
                '48小时快速退款48小时快速退款'
            }
        },
        components: {
            'title-bar': titlebar
        },
        methods: {
            jump: function () {
                navigator.push({
                    url: self.hostUrl + 'register.js'
                })
            },
            pop:function () {
                navigator.pop()
            }
        }
    }
</script>