<template>
    <div class="wrapper">
        <div class="contentBox" :style="{top:top+'px',bottom:bottom+'px'}">
            <scroller class="scroller" show-scrollbar="false">
                <div class="dataBox">
                    <data :datas="datas" :merchant="text"></data>
                </div>
                <div class="listBox" v-if="!isNull">
                    <waybill :waybill="waybill" :moneyData="moneyData" @change="change" @changeAll="changeAll" :allchoice="allchoice"></waybill>
                </div>
            </scroller>
            <div  v-if="isNull" class="null">
                <image :src="nullUrl" resize="contain" class="nullimage"></image>
                <text class="nulltext">暂无相关信息</text>
            </div>
            <filtrates class="filtrate" :title="pause" @fresh="fresh" @getID="getId"></filtrates>
            <title-bar :head="title" class="title"></title-bar>
            <result v-if="isOpacity" class="result" :result="result" @restoreRefresh='restoreRefresh' :style="{opacity:isOpacity? 1 : 0}"></result>
        </div>
        <div class="topBox" :style="{height:top+'px'}"></div>
    </div>
</template>

<script>
    import self from './self.config';
    import titleBar from './components/titleBar.vue';
    import search from './components/search.vue';
    import datas from './components/data.vue';
    import filtrates from './components/filtrate.vue';
    import waybill from './components/waybill.vue';
    import result from './components/result.vue';

    const stream = weex.requireModule("stream");
    const navigator = weex.requireModule("navigator");
    const storage = weex.requireModule("storage");
    const modal = weex.requireModule('modal');
    const animation = weex.requireModule("animation");

    const id = weex.config.bundleUrl.split("id=")[1];
    // const id = "20229";

    const get = function (selfsort,pause_category,call) {
        storage.getItem("token", function (e) {
            stream.fetch({
                method: "get",
                url: self.devUrl + "api/company/arrear/merchants/" + id + "/paused/delivery_sheets?include=pauseReasons&sort="+selfsort+"&pause_category[]="+pause_category,
                type: "json",
                headers: {
                    "Accept": "application/vnd.logistic.v3+json",
                    "Authorization": "bearer " + e.data
                }
            }, res=>{
                console.log(res);
                if(res.status == "401"){
                    weex.requireModule("nativeModule").refreshToken(function callback(value) {
                        storage.setItem("token",value.token);
                        stream.fetch({
                            method: "get",
                            url: self.devUrl + "api/company/arrear/merchants/" + id + "/paused/delivery_sheets?include=pauseReasons&sort="+selfsort+"&pause_category[]="+pause_category,
                            type: "json",
                            headers: {
                                "Accept": "application/vnd.logistic.v3+json",
                                "Authorization": "bearer " + value.data
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
    }
    export default {
        name: "index",
        data() {
            return {
                title: '',
                pause: "暂停分类",
                top:"40",
                bottom:"0",
                refreshing: false,
                isNull:false,
                allchoice:false,
                nullUrl:self.imgUrl + "img_kong.png",
                choiceUrl: self.imgUrl + 'icon_choice_normal.png',
                datas: {
                    total_reached_receivable_freight: 0,
                    total_receivable_goods_expense: 0,
                    total_merchants: 0,
                    amount: 0
                },
                isOpacity: false,
                text: "运单",
                waybill: [],
                moneyData:[],
                result: {
                    sum: 0,
                    freight: 0,
                    payment: 0,
                    num: 0
                },
                selfsort:"sign_time_desc",
                pause_category:-1
            }
        },
        created: function () {
            const _self = this;
            const platform = weex.config.env.platform;
            if(platform == "android"){
                _self.top = "0";
            }
            const info = weex.config.env.deviceModel;
            if(info === 'iPhone10,3' || info === 'iPhone10,6'){
                _self.top = "88";
                _self.bottom = "68";
            }
            storage.getItem("ids",function (e) {
                if(e.result == "success"){
                    get(_self.selfsort,e.data, res => {
                        const data = res.data;
                        if(data.data.length == 0){
                            _self.datas.total_reached_receivable_freight = 0;
                            _self.datas.total_receivable_goods_expense =0;
                            _self.datas.total_merchants = 0;
                            _self.datas.amount = 0;
                            _self.isNull = true;
                        }else{
                            _self.title = data.data[0].receiver;
                            _self.datas.total_reached_receivable_freight = data.meta.statistics.reached_receivable_freight;
                            _self.datas.total_receivable_goods_expense = data.meta.statistics.receivable_goods_expense;
                            _self.datas.total_merchants = data.meta.statistics.total_sheet;
                            _self.datas.amount = data.meta.statistics.amount;
                            for (let i in data.data) {
                                data.data[i].isActive = false;
                            }
                            _self.waybill = data.data;
                            _self.moneyData = data.meta.pause_statistics;
                        }
                    });
                }else{
                    get(_self.selfsort,_self.pause_category, res => {
                        const data = res.data;
                        if(data.data.length == 0){
                            _self.datas.total_reached_receivable_freight = 0;
                            _self.datas.total_receivable_goods_expense =0;
                            _self.datas.total_merchants = 0;
                            _self.datas.amount = 0;
                            _self.isNull = true;
                        }else{
                            _self.title = data.data[0].receiver;
                            _self.datas.total_reached_receivable_freight = data.meta.statistics.reached_receivable_freight;
                            _self.datas.total_receivable_goods_expense = data.meta.statistics.receivable_goods_expense;
                            _self.datas.total_merchants = data.meta.statistics.total_sheet;
                            _self.datas.amount = data.meta.statistics.amount;
                            for (let i in data.data) {
                                data.data[i].isActive = false;
                            }
                            _self.waybill = data.data;
                            _self.moneyData = data.meta.pause_statistics;
                        }
                    });
                }
            });
        },
        methods: {
            showresult: function () {
                let no = 0;
                for (let i in this.waybill) {
                    if (this.waybill[i].isActive) {
                        no++;
                    }
                }
                if (no == 0) {
                    this.isOpacity = false;
                } else {
                    this.isOpacity = true;
                }
            },
            change: function (index) {
                this.waybill[index].isActive = !this.waybill[index].isActive;
                let no = 0;
                for (let i in this.waybill) {
                    if (this.waybill[i].isActive) {
                        no++;
                    }
                }
                if (no == this.waybill.length) {
                    this.allchoice = true;
                } else {
                    this.allchoice = false;
                }
                this.sum();
                this.showresult();
            },
            changeAll: function () {
                this.allchoice = !this.allchoice;
                if (this.allchoice) {
                    for (let i in this.waybill) {
                        this.waybill[i].isActive = true;
                    }
                    this.sum();
                    this.isOpacity = true;
                } else {
                    for (let i in this.waybill) {
                        this.waybill[i].isActive = false;
                    }
                    this.initialize();
                    this.isOpacity = false;
                }

            },
            sum: function () {
                let freight = 0;
                let payment = 0;
                let num = 0;
                for (let i in this.waybill) {
                    if (this.waybill[i].isActive) {
                        num++;
                        freight += this.moneyData[i].reached_receivable_freight;
                        payment += this.moneyData[i].receivable_goods_expense;
                    }
                }
                this.result.freight = freight.toFixed(2);
                this.result.payment = payment.toFixed(2);
                this.result.sum = (freight + payment).toFixed(2);
                this.result.num = num;
            },
            initialize: function () {
                this.result.freight = 0;
                this.result.payment = 0;
                this.result.sum = 0;
                this.result.num = 0;
            },
            fresh: function (str) {
                const _self = this;
                _self.allchoice = false;
                _self.isOpacity = false;
                _self.selfsort = str;
                get(_self.selfsort,_self.pause_category, res => {
                    const data = res.data;
                    if(data.data.length == 0){
                        _self.datas.total_reached_receivable_freight = 0;
                        _self.datas.total_receivable_goods_expense =0;
                        _self.datas.total_merchants = 0;
                        _self.datas.amount = 0;
                        _self.isNull = true;
                    }else{
                        _self.isNull = false;
                        _self.title = data.data[0].receiver;
                        _self.datas.total_reached_receivable_freight = data.meta.statistics.reached_receivable_freight;
                        _self.datas.total_receivable_goods_expense = data.meta.statistics.receivable_goods_expense;
                        _self.datas.total_merchants = data.meta.statistics.total_sheet;
                        _self.datas.amount = data.meta.statistics.amount;
                        for (let i in data.data) {
                            data.data[i].isActive = false;
                        }
                        _self.waybill = data.data;
                        _self.moneyData = data.meta.pause_statistics;
                    }
                })
            },
            getId: function (ids) {
                const _self = this;
                _self.allchoice = false;
                _self.isOpacity = false;
                _self.pause_category = ids;
                get(_self.selfsort,_self.pause_category, res => {
                    const data = res.data;
                    if(data.data.length == 0){
                        _self.datas.total_reached_receivable_freight = 0;
                        _self.datas.total_receivable_goods_expense =0;
                        _self.datas.total_merchants = 0;
                        _self.datas.amount = 0;
                        _self.isNull = true;
                    }else{
                        _self.isNull = false;
                        _self.title = data.data[0].receiver;
                        _self.datas.total_reached_receivable_freight = data.meta.statistics.reached_receivable_freight;
                        _self.datas.total_receivable_goods_expense = data.meta.statistics.receivable_goods_expense;
                        _self.datas.total_merchants = data.meta.statistics.total_sheet;
                        _self.datas.amount = data.meta.statistics.amount;
                        for (let i in data.data) {
                            data.data[i].isActive = false;
                        }
                        _self.waybill = data.data;
                        _self.moneyData = data.meta.pause_statistics;
                    }
                })
            },
            restoreRefresh: function () {
                const _self = this;
                _self.isOpacity = false;
                var devliery_sheet_nos = [];
                for (let i in _self.waybill) {
                    if (_self.waybill[i].isActive) {
                        devliery_sheet_nos.push(_self.waybill[i].no)
                    }
                }
               storage.getItem("token",function(e){
                   stream.fetch({
                       method: 'POST',
                       type: 'json',
                       url: self.devUrl + 'api/restored/delivery_sheets',
                       body: "no=" + devliery_sheet_nos.join(","),
                       type: "json",
                       headers: {
                           "Accept": "application/vnd.logistic.v3+json",
                           "Content-Type": "application/x-www-form-urlencoded",
                           "Authorization": "bearer " + e.data
                       },
                   }, res => {
                       if (res.status == "204") {
                           get(_self.selfsort,_self.pause_category, res => {
                               if(res.data.data.length == 0){
                                   _self.datas.total_reached_receivable_freight = 0;
                                   _self.datas.total_receivable_goods_expense =0;
                                   _self.datas.total_merchants = 0;
                                   _self.datas.amount = 0;
                                   _self.waybill = [];
                                   _self.isOpacity = false;
                                   _self.isNull = true;
                               }else {
                                   const data = res.data;
                                   _self.datas.total_reached_receivable_freight = data.meta.statistics.reached_receivable_freight;
                                   _self.datas.total_receivable_goods_expense = data.meta.statistics.receivable_goods_expense;
                                   _self.datas.total_merchants = data.meta.statistics.total_sheet;
                                   _self.datas.amount = data.meta.statistics.amount;
                                   for (let i in data.data) {
                                       data.data[i].isActive = false;
                                   }
                                   _self.waybill = data.data;
                                   _self.moneyData = data.meta.pause_statistics;
                               }
                           })
                       }
                   })
               })
            },
            onref: function () {
                const _self = this;
                this.refreshing = true;
                setTimeout(() => {
                    this.refreshing = false
                }, 1000)
                get(_self.selfsort,_self.pause_category, res => {
                    const data = res.data;
                    if(data.data.length == 0){
                        _self.datas.total_reached_receivable_freight = 0;
                        _self.datas.total_receivable_goods_expense =0;
                        _self.datas.total_merchants = 0;
                        _self.datas.amount = 0;
                        _self.isNull = true;
                    }else{
                        _self.isNull = false;
                        _self.title = data.data[0].receiver;
                        _self.datas.total_reached_receivable_freight = data.meta.statistics.reached_receivable_freight;
                        _self.datas.total_receivable_goods_expense = data.meta.statistics.receivable_goods_expense;
                        _self.datas.total_merchants = data.meta.statistics.total_sheet;
                        _self.datas.amount = data.meta.statistics.amount;
                        for (let i in data.data) {
                            data.data[i].isActive = false;
                        }
                        _self.waybill = data.data;
                        _self.moneyData = data.meta.pause_statistics;
                    }
                });
            },
        },
        components: {
            'title-bar': titleBar,
            'search': search,
            'data': datas,
            'filtrates': filtrates,
            'waybill': waybill,
            'result': result
        }
    }
</script>

<style scoped>
    .wrapper {
        align-items: center;
        background-color: #f3f3f3;
        position: absolute;
        top: 0;
        bottom: 0;
        width: 750px;
    }
    .contentBox{
        align-items: center;
        position: absolute;
        background-color: #f3f3f3;
        width: 750px;
    }
    .topBox{
        width: 750px;
        background-color: #3c98e6;
    }
    .title {
        position: absolute;
        top: 0;
    }
    .null{
        width: 750px;
        align-items: center;
        position: absolute;
        top: 610px;
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
        position: absolute;
        top: 128px;
        width: 750px;
        background-color: #fff;
        padding-top: 16px;
        align-items: center;
        padding-bottom: 16px;
    }

    .refresh {
        align-items: center;
    }

    .indicator {
        margin-top: 15px;
        margin-bottom: 15px;
        width: 40px;
        height: 40px;
        color: #0088fb;
    }

    .dataBox {
        width: 750px;
        background-color: #fff;
        border-bottom-color: #dddddd;
        border-bottom-width: 1px;
        align-items: center;
    }

    .scroller {
        width: 750px;
        position: absolute;
        top: 168px;
        bottom: 0;
    }

    .filtrate {
        position: absolute;
        top: 88px;
    }

    .listBox {
        align-items: flex-end;
        background-color: #fff;
        margin-top: 30px;
        margin-bottom: 96px;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
        border-top-width: 1px;
        border-top-color: #dddddd;
    }

    .result {
        position: absolute;
        bottom: 0;
    }
</style>