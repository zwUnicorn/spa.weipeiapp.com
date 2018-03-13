<template>
    <div class="wrapper">
        <title-bar :head="title" :subhead="subtitle" class="title"></title-bar>
        <list class="list">
            <refresh class="refresh" @refresh="onRefresh" :display="refreshing ? 'show' : 'hide'">
                <image class="load" :src="loadUrl" resize="contain" ref="load"></image>
            </refresh>
            <cell class="cell" v-for="item in info">
                <info-bar :info="item" @click.native="jump(item.order_no)"></info-bar>
            </cell>
        </list>
    </div>
</template>

<style scoped>
    .list {
        position: fixed;
        width: 750px;
        top: 128px;
        bottom: 0;
        background-color: #dddddd;
    }

    .title {
        position: fixed;
        top: 0;
    }

    .cell {
        background-color: #fff;
    }
    .refresh{
        align-items: center;
        justify-content: center;
        padding-bottom: 15px;
        padding-top: 15px;
        width: 750px;
    }
    .load{
        width: 39px;
        height: 38px;
    }
</style>

<script>
    import self from './self.config'
    import titlebar from './components/titlebar.vue'
    import infobar from './components/infobar.vue'

    const navigator = weex.requireModule('navigator');
    const stream = weex.requireModule('stream');
    const storage = weex.requireModule('storage');
    const animation = weex.requireModule('animation');

    export default {
        data() {
            return {
                title: '历史白条',
                subtitle: '',
                logoUrl: self.imgUrl + 'img_open.png',
                info: [],
                message: '2',
                refreshing:false,
                loadUrl:self.imgUrl + 'loading_gray.png',
                deg:720,
            }
        },
        mounted: function () {
            const _self = this;
            storage.getItem("token", function (e) {
                stream.fetch({
                    method: 'get',
                    url: self.devUrl + 'v3/white-finance?is_single=true&status=',
                    type: 'json',
                    headers: {
                        'Content-Type': 'application/json',
                        'accept': 'application/vnd.api.v3+json',
                        'authorization': 'bearer ' + e.data
                    }
                }, res => {
                    const data = res.data.data;
                    for (let i = 0; i < data.length; i++) {
                        _self.info.unshift(data[i]);
                    }
                    for(let i = 0;i< _self.info.length;i++){
                        if ( _self.info[i].status == 'billing') {
                            _self.info[i].status = '待出款'
                        } else if ( _self.info[i].status == 'billed') {
                            _self.info[i].status = '待还款'
                        } else if ( _self.info[i].status == 'paid_off' ||  _self.info[i].status == 'paying') {
                            _self.info[i].status = '已还款'
                        } else if ( _self.info[i].status == 'canceled') {
                            _self.info[i].status = '已取消'
                        }
                    }
                })
            })

        },
        components: {
            'title-bar': titlebar,
            'info-bar': infobar
        },
        methods: {
            jump: function (order_id) {
                navigator.push({
                    url:self.hostUrl+'detailsInfo.js?id='+order_id
                })
            },
            onRefresh:function () {
                const _self = this;
                const loadEl = this.$refs.load;
                animation.transition(loadEl, {
                    styles: {
                        transform:'rotate('+ _self.deg +'deg)'
                    },
                    duration:1500
                });
                _self.deg += 720;
                storage.getItem("token", function (e) {
                    stream.fetch({
                        method: 'get',
                        url: self.devUrl + 'v3/white-finance?is_single=true&status=',
                        type: 'json',
                        headers: {
                            'Content-Type': 'application/json',
                            'accept': 'application/vnd.api.v3+json',
                            'authorization': 'bearer ' + e.data
                        }
                    }, res => {
                        const data = res.data.data;
                        _self.info = [];
                        for (let i = 0; i < data.length; i++) {
                            _self.info.unshift(data[i]);
                        }
                        for(let i = 0;i< _self.info.length;i++){
                            if ( _self.info[i].status == 'billing') {
                                _self.info[i].status = '待出款'
                            } else if ( _self.info[i].status == 'billed') {
                                _self.info[i].status = '待还款'
                            } else if ( _self.info[i].status == 'paid_off' ||  _self.info[i].status == 'paying') {
                                _self.info[i].status = '已还款'
                            } else if ( _self.info[i].status == 'canceled') {
                                _self.info[i].status = '已取消'
                            }
                        }
                    })
                })
                this.refreshing = true;
                setTimeout(() => {
                    this.refreshing = false
                }, 1000)
            }
        }
    }
</script>