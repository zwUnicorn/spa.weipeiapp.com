<template>
    <div class="wrapper">
        <title-bar :head="title" :subhead="subtitle"></title-bar>
        <div class="message" :style="{opacity:opacity}">
            <image class="messageIcon" :src="messageIcon" resize="contain"></image>
            <text class="messageInfo">{{message}}</text>
        </div>
        <scroller class="scroller">
            <image :src="logoUrl" class="logo" resize="cover"></image>
            <div class="button" @click="jump">
                <text class="text">开通</text>
            </div>
        </scroller>
    </div>
</template>
<style scoped>
    .wrapper {
        align-items: center;
    }

    .logo {
        width: 750px;
        height: 1134px;
    }
    .message {
        width: 750px;
        height: 70px;
        flex-direction: row;
        align-items: center;
        background-color: #fff4d7;
        padding-left: 32px;
        position: fixed;
        top: 128px;
    }

    .messageIcon {
        width: 36px;
        height: 36px;
    }

    .messageInfo {
        font-size: 26px;
        margin-left: 16px;
        color: #9d762a;
    }
    .scroller {
        width: 750px;
        justify-content: center;
    }

    .button {
        width: 590px;
        height: 80px;
        background-color: #44b3e1;
        font-size: 28px;
        align-items: center;
        justify-content: center;
        border-radius: 10px;
        position: absolute;
        bottom: 50px;
        margin-left: 80px;
    }

    .text {
        color: #fff;
        font-size: 34px;
    }

</style>
<script>
    import self from './self.config'
    import titlebar from './components/titlebar.vue'

    const navigator = weex.requireModule('navigator')
    const stream = weex.requireModule('stream')
    const storage = weex.requireModule('storage')

    export default {
        data() {
            return {
                title: '维配白条',
                subtitle: '',
                logoUrl: self.imgUrl + 'img_open.png',
                messageIcon: self.imgUrl + 'icon_exc.png',
                opacity: 0,
                message:'权限不足,请联系管理员!!!'
            }
        },
        mounted: function () {
            weex.requireModule('nativeModule').getToken(function callback(res) {
                storage.setItem("token", res.token);
            })
        },
        components: {
            'title-bar': titlebar
        },
        methods: {
            jump: function () {
                const _self = this;
                storage.getItem("token",function (e) {
                    stream.fetch({
                        methods:'get',
                        url: self.devUrl+'v3/white-finance/unauthorized/administrator',
                        type:'json',
                        headers:{
                            'Content-Type':'application/x-www-form-urlencoded',
                            'accept':'application/vnd.api.v3+json',
                            'authorization':'bearer '+ e.data
                        }
                    },res=>{
                        if(res.data.status_code == '0'){
                            navigator.push({
                                url: self.hostUrl + 'agreement.js'
                            })
                        }else{
                            _self.opacity = 1;
                            setTimeout(function () {
                                _self.opacity = 0
                            }, 2000)
                        }
                    })
                })
            }
        }
    }

</script>
