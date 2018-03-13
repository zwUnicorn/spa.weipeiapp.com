<template>
    <div class="wrapper">
        <div v-if="isNull" class="null">
            <image :src="nullUrl" resize="contain" class="nullimage"></image>
            <text class="nulltext">暂无相关信息</text>
        </div>
        <list class="list" :style="{top:listTop+'px'}" ref="list">
            <refresh class="refresh" @refresh="onrefresh" :display="refreshing ? 'show' : 'hide'">
                <loading-indicator class="indicator"></loading-indicator>
            </refresh>
            <cell>
                <div class="dataBox">
                    <data :datas="datas" :merchant="text"></data>
                </div>
                <div class="dote"></div>
            </cell>
            <cell class="listBox" v-if="!isNull">
                <merchant :merchant="merchants"></merchant>
            </cell>
            <loading v-if="isLoading" class="loading" @loading="onloading" :display="loadinging ? 'show' : 'hide'">
                <text class="indicator-text">加载中 ...</text>
                <loading-indicator class="indicator"></loading-indicator>
            </loading>
        </list>
        <filtrate class="filtrate" :station_s="station_s" :pauses="pauses" @fresh="fresh" @get_station="get_station" @getID="getId" @getNoID="getNoID" :style="{top:height+'px'}"></filtrate>
        <div ref="header" class="header">
            <div class="topBox" :style="{height:top+'px'}"></div>
            <title-bar :head="title" class="title"></title-bar>
            <div class="box">
                <search @serch="serch"></search>
            </div>
        </div>
    </div>

</template>

<script>
    import titleBar from './components/titleBar.vue';
    import search from './components/search.vue';
    import datas from './components/data.vue';
    import filtrate from './components/filtrate1.vue';
    import merchant from './components/merchant.vue';
    import self from './self.config';

    const stream = weex.requireModule("stream");
    const navigator = weex.requireModule("navigator");
    const modal = weex.requireModule("modal");
    const storage = weex.requireModule("storage");
    const dom = weex.requireModule("dom");

    const get = function (sort,id,keyword,page,pause_category,call) {
        weex.requireModule("nativeModule").getToken(function callback(res) {
            storage.setItem("token", res.token);
            stream.fetch({
                method: "get",
                url: self.devUrl + "api/company/arrear/merchants?amount_sort="+sort+id+"&keyword="+encodeURI(keyword)+"&page="+page+"&pause_category[]="+pause_category,
                type: "json",
                headers: {
                    "Accept": "application/vnd.logistic.v3+json",
                    "Authorization": "bearer "+res.token
                }
            },res=>{
                if(res.status == "401"){
                    weex.requireModule("nativeModule").refreshToken(function callback(value) {
                        storage.setItem("token",value.token);
                        stream.fetch({
                            method: "get",
                            url: self.devUrl + "api/company/arrear/merchants?amount_sort="+sort+id+"&keyword="+encodeURI(keyword)+"&page="+page+"&pause_category[]="+pause_category,
                            type: "json",
                            headers: {
                                "Accept": "application/vnd.logistic.v3+json",
                                "Authorization": "bearer " + value.token
                            }
                        },res=>{
                            call(res);
                        });
                    })
                }else{
                    call(res);
                }

            });
        })
    };

    export default {
        name: "index",
        data() {
            return {
                title: '欠费客户',
                text: "商户",
                top:"40",
                station_s: '选择站点',
                pauses:'暂停分类',
                nullUrl:self.imgUrl + "img_kong.png",
                datas: {
                    total_merchants:0,
                    total_reached_receivable_freight:0,
                    total_receivable_goods_expense:0,
                    amount:0
                },
                merchants: [],
                refreshing: false,
                isNull:false,
                isText:false,
                sort:"",
                stations:"",
                keyword:"",
                page:1,
                pause_category:-1,
                showLoading:"hide",
                loadinging:false,
                isLoading:false,
                height:0,
                listTop:0
            }
        },
        created: function () {
            const _self = this;
            get(_self.sort,_self.stations,_self.keyword,_self.page,_self.pause_category,res=>{
                _self.dataProcessing(res)
            });
            storage.removeItem("pauseValue");
            storage.removeItem("ids");
            const platform = weex.config.env.platform;
            if(platform == "android"){
                _self.top = "0";
            }
            const info = weex.config.env.deviceModel;
            if(info === 'iPhone10,3' || info === 'iPhone10,6'){
                _self.top = "88";
            }

        },
        mounted:function () {
            const _self = this;
            setTimeout(function () {
                dom.getComponentRect(_self.$refs.header,res=>{
                    _self.height = res.size.height;
                    _self.listTop = res.size.height + 80;
                });
            },50);
        },
        methods: {
            //数据处理
            dataProcessing(res){
                const _self = this;
                const data = res.data;
                if(data.data.length == 0){
                    _self.datas.total_merchants = 0;
                    _self.datas.total_reached_receivable_freight = 0;
                    _self.datas.total_receivable_goods_expense = 0;
                    _self.datas.amount = 0;
                    _self.isNull = true;
                    _self.isLoading = false;
                }else{
                    _self.isNull = false;
                    _self.datas = data.meta.statistics;
                    _self.merchants = data.data;
                    if(data.data.length == 15){
                        _self.isLoading =  true;
                    }else{
                        _self.isLoading = false;
                    }
                }
            },
            onloading () {
                const _self = this;
                _self.loadinging = true;
                setTimeout(() => {
                    _self.loadinging = false;
                    _self.page++;
                    get(_self.sort,_self.stations,_self.keyword,_self.page,_self.pause_category,res=>{
                        const data = res.data;
                        if(_self.merchants.length == 0){
                            _self.isNull = true;
                        }else{
                            _self.isNull = false;
                        }
                        _self.datas = data.meta.statistics;
                        if(data.data.length != 0){
                            for(let i in data.data){
                                _self.merchants.push(data.data[i]);
                            }
                            if(data.data.length == 15){
                                _self.isLoading = true;
                            }else{
                                _self.isLoading = false;
                            }
                        }
                    })
                }, 1000)
            },
            get_station: function (stations) {
                const _self = this;
                _self.stations="";
                _self.page = 1;
                for(let i in stations){
                    _self.stations+=("&station_id[]="+stations[i]);
                }
                get(_self.sort,_self.stations,_self.keyword,_self.page,_self.pause_category,res=>{
                    _self.dataProcessing(res)
                })
            },
            getId: function (ids,value) {
                const _self = this;
                storage.setItem("pauseValue",value);
                _self.page = 1;
                _self.pause_category = ids;
                get(_self.sort,_self.stations,_self.keyword,_self.page,_self.pause_category,res=>{
                    _self.dataProcessing(res)
                });
            },
            fresh: function (str) {
                const _self = this;
                _self.sort = str;
                _self.page = 1;
                get(_self.sort,_self.stations,_self.keyword,_self.page,_self.pause_category,res=>{
                    _self.dataProcessing(res)
                })
            },
            onrefresh:function () {
                const _self = this;
                _self.refreshing = true;
                _self.page = 1;
                get(_self.sort,_self.stations,_self.keyword,_self.page,_self.pause_category,res=>{
                    _self.dataProcessing(res)
                })
                setTimeout(() => {
                    _self.refreshing = false
                }, 1000)
            },
            serch:function (msg) {
                const _self = this;
                _self.keyword = msg;
                _self.page=1;
                get(_self.sort,_self.stations,_self.keyword,_self.page,_self.pause_category,res=>{
                    _self.dataProcessing(res)
                })
            }
        },
        components: {
            'title-bar': titleBar,
            'search': search,
            'data': datas,
            'filtrate': filtrate,
            'merchant': merchant,
        }
    }
</script>

<style scoped>
    .wrapper {
        align-items: center;
        position: absolute;
        top: 0;
        bottom: 0;
        background-color: #f3f3f3;
        width: 750px;
    }
    .list{
        width: 750px;
        position: absolute;
        bottom: 0;
    }
    .header{
        width: 750px;
        position: absolute;
        top: 0;
    }
    .refresh{
        width: 750px;
        align-items: center;
    }
    .indicator-text {
        color: #888888;
        font-size: 32px;
        text-align: center;
    }
    .contentBox{
        align-items: center;
        position: absolute;
        bottom: 0;
        background-color: #f3f3f3;
        width: 750px;
    }
    .topBox{
        width: 750px;
        background-color: #3c98e6;
    }
    .title{

    }
    .null{
        width: 750px;
        align-items: center;
        position: absolute;
        top: 512px;
    }
    .nullimage{
        width: 214px;
        height: 254px;
    }
    .nulltext{
        font-size: 30px;
        color: #999;
    }
    .box {
        width: 750px;
        background-color: #fff;
        padding-top: 16px;
        padding-bottom: 16px;
    }
    .loading{
        width: 750px;
        align-items: center;
    }
    .load{
        color: #999999;
    }
    .filtrate{
        align-items: center;
        position: absolute;
    }
    .dataBox {
        width: 750px;
        background-color: #fff;
        border-bottom-color: #dddddd;
        border-bottom-width: 1px;
        align-items: center;
    }
    .indicator{
        color: #aaaaaa;
        width: 40px;
        height: 40px;
        margin-top: 20px;
        margin-bottom: 20px;
    }
    .dote{
        width: 750px;
        height: 30px;
    }
    .listBox {
        align-items: flex-end;
        background-color: #fff;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
        border-top-width: 1px;
        border-top-color: #dddddd;
    }
</style>