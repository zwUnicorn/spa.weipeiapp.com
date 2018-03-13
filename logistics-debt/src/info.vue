<template>
    <div class="wrapper" @swipe="swipe">
        <div class="contentBox" :style="{top:top+'px',bottom:bottom+'px'}">
            <scroller class="scroller" show-scrollbar="false">
                <refresh v-if="isRefresh" class="refresh" @refresh="onrefresh" :display="refreshing ? 'show' : 'hide'">
                    <loading-indicator class="indicator"></loading-indicator>
                </refresh>
                <div class="dataBox">
                    <data :title="text" :total="newdata.totals" :amount="newdata.amounts" :back_freight="newdata.back_freight" :unreach="newdata.unreach"></data>
                </div>
                <div class="listBox" v-if="!isNull">
                    <waybill :waybill="waybill" @change="change" @changeAll="changeAll" :allchoice="allchoice" :isClick="isClick"></waybill>
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

    const id = weex.config.bundleUrl.split("id=")[1];
    // const id = "20229";

    const get = function (sort,call) {
        storage.getItem("token", function (e) {
            stream.fetch({
                method: "get",
                url: self.devUrl + "api/company/merchants/"+id+"/monthly_settlement/delivery_sheets?amount_sort="+sort,
                type: "json",
                headers: {
                    "Accept": "application/vnd.logistic.v3+json",
                    "Authorization": "bearer " + e.data
                }
            }, res=>{
                call(res);
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
                isClick:true,
                nullUrl:self.imgUrl + "img_kong.png",
                choiceUrl: self.imgUrl + 'icon_choice_normal.png',
                newdata:{
                    totals:0,
                    amounts:0,
                    back_freight:0,
                    unreach:0
                },
                isOpacity: false,
                text: "运单",
                waybill: [],
                result: {
                    sum: 0,
                    num: 0
                },
                selfsort:"sign_time_desc",
                pause_category:-1,
                isRefresh:false
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
            get(_self.selfsort, res => {
                _self.dataProcessing(res);
            });
        },
        methods: {
            swipe(event){
                // modal.alert({
                //     message:'666'
                // })
            },
            //数据处理
            dataProcessing(res){
                const _self = this;
                const data = res.data;
                if(data.data.length == 0){
                    self.newdata.totals = 0;
                    _self.newdata.amounts = 0;
                    _self.newdata.unreach = 0;
                    _self.newdata.back_freight = 0;
                    _self.isNull = true;
                }else{
                    _self.title = data.data[0].sender;
                    let back = 0;
                    let unreach = 0;
                    let num=0;
                    for (let i in data.data) {
                        data.data[i].isActive = false;
                        data.data[i].isChoice = false;
                        if(!data.data[i].can_create_cashback_sheet){
                            num++;
                        }
                        back += data.data[i].back_freight;
                        unreach += data.data[i].unreach_receivable_freight;
                    }
                    if(num == data.data.length){
                        _self.isClick = false;
                    }
                    _self.waybill = data.data;
                    _self.newdata.totals  = data.meta.statistics.total_sheet;
                    _self.newdata.back_freight = back;
                    _self.newdata.amounts = data.meta.statistics.amount;
                    _self.newdata.unreach = unreach;
                }
            },
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
                let newarr = [];
                for (let i in this.waybill) {
                    if (this.waybill[i].isActive) {
                        no++;
                    }
                    if(!this.waybill[i].isChoice){
                        newarr.push(this.waybill[i]);
                    }
                }
                if (no == newarr.length) {
                    this.allchoice = true;
                } else {
                    this.allchoice = false;
                }
                this.sum();
                this.showresult();
            },
            changeAll: function () {
                const _self = this;
                let no = 0;
                this.allchoice = !this.allchoice;
                if (this.allchoice) {
                    for (let i in _self.waybill) {
                        if(_self.waybill[i].can_create_cashback_sheet){
                            _self.waybill[i].isActive = true;
                        }else{
                            no++;
                            _self.waybill[i].isActive = false;
                        }
                    }
                    this.sum();
                    if(no == _self.waybill.length){
                        this.isOpacity = false;
                    }else{
                        this.isOpacity = true;
                    }
                } else {
                    for (let i in this.waybill) {
                        if(!_self.waybill[i].isChoice){
                            this.waybill[i].isActive = false;
                        }
                    }
                    this.initialize();
                    this.isOpacity = false;
                }

            },
            sum: function () {
                let num = 0;
                let sum = 0;
                for (let i in this.waybill) {
                    if (this.waybill[i].isActive) {
                        num++;
                         sum += this.waybill[i].can_cashback_amount;
                    }
                }
                this.result.sum = sum;
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
                get(_self.selfsort, res => {
                    _self.dataProcessing(res);
                })
            },
            restoreRefresh: function () {
                const _self = this;
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
                       url: self.devUrl + 'api/monthly_settlement/cash_back_sheets',
                       body: {
                           "sheet_nos": devliery_sheet_nos
                       },
                       type: "json",
                       headers: {
                           "Accept": "application/vnd.logistic.v3+json",
                           "Content-Type": "application/json",
                           "Authorization": "bearer " + e.data
                       },
                   }, res => {
                       if(res.ok){
                           get(_self.selfsort, res => {
                               _self.dataProcessing(res);
                           });
                           _self.isOpacity = false;
                           _self.allchoice = false;
                           weex.requireModule("nativeModule").redirectReturnBillDetail(JSON.stringify(res.data.data),function callback() {
                               get(_self.selfsort, res => {
                                   _self.dataProcessing(res);
                               });
                               _self.isOpacity = false;
                               _self.allchoice = false;
                           })
                       }else{
                           modal.alert({
                               message:res.message
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
        background-color: #007bff;
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