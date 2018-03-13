<template>
    <div class="wrapper">
        <title-bar :head="title" :subhead="subtitle"></title-bar>
        <div class="message" :style="{opacity:opacity}">
            <image class="messageIcon" :src="messageIcon" resize="contain"></image>
            <text class="messageInfo">{{message}}</text>
        </div>
        <div class="form">
            <div class="inputBox">
                <image :src="nameIcon" class="image" resize="contain"></image>
                <input type="text" class="input" placeholder="请输入姓名" v-model="name">
            </div>
            <div class="inputBox">
                <image :src="idIcon" class="image" resize="contain"></image>
                <input type="number" class="input" placeholder="请输入身份证号码" v-model="idCard">
            </div>
        </div>
        <text class="text" @click="jump">确认开通</text>
    </div>
</template>

<style scoped>
    .wrapper {
        align-items: center;
    }

    .form {
        padding-top: 60px;
    }

    .inputBox {
        width: 590px;
        height: 120px;
        border-bottom-width: 1px;
        border-bottom-color: #cccccc;
        align-items: center;
        position: relative;
    }

    .input {
        margin-left: 24px;
        font-size: 28px;
        color: #333333;
        height: 60px;
        position: absolute;
        left: 84px;
        bottom: 12px;
        width: 506px;
        placeholder-color: #cccccc;
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

    .image {
        width: 60px;
        height: 60px;
        position: absolute;
        left: 0;
        bottom: 12px;
    }

    .text {
        width: 590px;
        height: 80px;
        text-align: center;
        line-height: 80px;
        margin-top: 64px;
        color: #ffffff;
        font-size: 28px;
        background-color: #44b3e1;
        border-radius: 10px;
    }
</style>

<script>
    import self from './self.config'
    import titlebar from './components/titlebar.vue'

    const navigator = weex.requireModule('navigator');
    const stream = weex.requireModule('stream');
    const storage = weex.requireModule('storage');

    export default {
        data: {
            title: '开通维配白条',
            subtitle: '',
            nameIcon: self.imgUrl + 'icon_nameinput.png',
            idIcon: self.imgUrl + 'icon_numinput.png',
            messageIcon: self.imgUrl + 'icon_exc.png',
            message: '开通失败，请检查输入',
            opacity: 0,
            name: '',
            idCard: ''
        },
        components: {
            'title-bar': titlebar
        },
        methods: {
            jump: function () {
                const _self = this
                if (_self.name == '' || _self.idCard == '') {
                    _self.opacity = 1;
                    _self.message = '信息未输入完整，请重新输入';
                    setTimeout(function () {
                        _self.opacity = 0
                    }, 2000)
                } else {
                    storage.getItem("token",function (e) {
                        stream.fetch({
                            method: 'post',
                            url: self.devUrl + 'v3/white-finance/account',
                            type: 'json',
                            headers: {
                                'Content-Type': 'application/json',
                                'accept': 'application/vnd.api.v3+json',
                                'authorization': 'Bearer '+e.data
                            },
                            body: {
                                id_card_number: _self.idCard,
                                realname: _self.name
                            }
                        }, res => {
                            if (res.data.status_code != '0') {
                                _self.opacity = 1;
                                _self.message = res.data.message;
                                setTimeout(function () {
                                    _self.opacity = 0
                                }, 2000)
                            } else {
                                navigator.push({
                                    url: self.hostUrl + 'personal.js'
                                })
                            }
                        })
                    })
                }
            }
        }
    }
</script>