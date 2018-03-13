<template>
    <div class="wrapper">
        <list class="list" :style="{top:listTop+'px'}" ref="list">
            <refresh class="refresh" @refresh="onrefresh" :display="refreshing ? 'show' : 'hide'">
                <loading-indicator class="indicator"></loading-indicator>
            </refresh>
            <cell>
                <div class="dataBox">
                    <data :title="text" :total="datas.total" :amount="datas.amount" :back_freight="datas.back_freight" :unreach="datas.unreach"></data>
                </div>
                <div class="dote"></div>
            </cell>
            <cell class="listBox" v-if="!isNull">
                <waybill :waybill="waybill" :allchoice="allchoice" :isClick="isClick" @change="change" @changeAll="changeAll"></waybill>
            </cell>
            <loading v-if="isLoading" class="loading" @loading="onloading" :display="loadinging ? 'show' : 'hide'">
                <text class="indicator-text">加载中 ...</text>
                <loading-indicator class="indicator"></loading-indicator>
            </loading>
        </list>
        <div v-if="isNull" class="null">
            <image :src="nullUrl" resize="contain" class="nullimage"></image>
            <text class="nulltext">暂无相关信息</text>
        </div>
        <filtrate class="filtrate" @fresh="fresh" :style="{top:height+'px'}"></filtrate>
        <div ref="header" class="header">
            <div class="topBox" :style="{height:top+'px'}"></div>
            <title-bar :head="title" class="title"></title-bar>
        </div>
        <result v-if="isOpacity" class="result" :result="result" @restoreRefresh='restoreRefresh' :style="{opacity:isOpacity? 1 : 0}"></result>
    </div>

</template>

<script>
    import titleBar from './components/titleBar.vue';
    import datas from './components/data.vue';
    import filtrate from './components/filtrate2.vue';
    import waybill from './components/waybill.vue';
    import self from './self.config';
    import result from './components/result.vue';

    const stream = weex.requireModule("stream");
    const navigator = weex.requireModule("navigator");
    const modal = weex.requireModule("modal");
    const storage = weex.requireModule("storage");
    const dom = weex.requireModule("dom");
    const id = weex.config.bundleUrl.split("id=")[1];

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
    };

    export default {
        name: "index",
        data() {
            return {
                title: '现付挂账',
                top:"40",
                text:'运单',
                nullUrl:self.imgUrl + "img_kong.png",
                waybill: [],
                allchoice:false,
                isClick:true,
                isOpacity:false,
                datas: {
                    total:0,
                    amount:0,
                    back_freight:0,
                    unreach:0
                },
                result: {
                    sum: 0,
                    num: 0
                },
                refreshing: false,
                isNull:false,
                sort:"",
                loadinging:false,
                isLoading:false,
                height:0,
                listTop:0,
                activeNum:0
            }
        },
        created: function () {
            const _self = this;
            get(_self.sort,res=>{
                _self.dataProcessing(res);
            });
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
            // while (_self.height == 0 ){
            //     dom.getComponentRect(_self.$refs.header,res=>{
            //         _self.height = res.size.height;
            //         _self.listTop = res.size.height + 80;
            //     });
            // }
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
                    self.datas.totals = 0;
                    _self.datas.amounts = 0;
                    _self.datas.unreach = 0;
                    _self.datas.back_freight = 0;
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
                    _self.activeNum = num;
                    _self.waybill = data.data;
                    _self.datas.total  = data.meta.statistics.total_sheet;
                    _self.datas.back_freight = back;
                    _self.datas.amount = data.meta.statistics.amount;
                    _self.datas.unreach = unreach;
                }
            },
            //上划加载
            onloading () {
                const _self = this;
                _self.loadinging = true;
                setTimeout(() => {
                    _self.loadinging = false;
                    _self.page++;
                    get(_self.sort,_self.stations,_self.keyword,_self.page,res=>{
                        const data = res.data;
                        if(_self.merchants.length == 0){
                            _self.isNull = true;
                        }else{
                            _self.isNull = false;
                        }
                        if(data.data.length != 0){
                            for(let i in data.data){
                                _self.merchants.push(data.data[i]);
                            }
                        }else{
                            _self.isLoading = false;
                        }
                    })
                }, 1000)
            },

            //金额排序筛选
            fresh: function (str) {
                const _self = this;
                _self.sort = str;
                get(_self.sort,res=>{
                    _self.dataProcessing(res);
                })
            },
            //下拉刷新
            onrefresh:function () {
                const _self = this;
                _self.refreshing = true;
                get(_self.sort,res=>{
                    _self.dataProcessing(res);
                });
                setTimeout(() => {
                    _self.refreshing = false
                }, 1000)
            },
            //单击选项
            change: function (index) {
                this.waybill[index].isActive = !this.waybill[index].isActive;
                let no = 0;
                let newarr = [];
                for (let i in this.waybill) {
                    if (this.waybill[i].isActive) {
                        no++;
                    }
                    if(this.waybill[i].can_cashback_amount != 0){
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
            //全选
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
        },
        components: {
            'title-bar': titleBar,
            'data': datas,
            'filtrate': filtrate,
            'waybill': waybill,
            'result' : result
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
    .topBox{
        width: 750px;
        background-color: #3c98e6;
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
    .indicator-text {
        color: #888888;
        font-size: 32px;
        text-align: center;
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
        /*background-color: #fff;*/
        margin-bottom: 98px;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
        border-top-width: 1px;
        border-top-color: #dddddd;
    }
    .null{
        width: 750px;
        align-items: center;
        position: absolute;
        top: 600px;
    }
    .nullimage{
        width: 214px;
        height: 254px;
    }
    .nulltext{
        font-size: 30px;
        color: #999;
    }
    .result {
        position: absolute;
        bottom: 0;
    }
</style>