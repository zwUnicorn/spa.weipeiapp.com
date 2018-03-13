<template>
    <div class="wrapper">
        <list class="list" :style="{top:listTop+'px'}" ref="list">
            <refresh class="refresh" @refresh="onrefresh" :display="refreshings ? 'show' : 'hide'">
                <loading-indicator class="indicator"></loading-indicator>
            </refresh>
            <cell>
                <div class="dataBox">
                    <data :title="text" :freight_amount="data.freight_amount" :goods_expense_amount="data.goods_expense_amount" :freight_cashbacked_amount="data.freight_cashbacked_amount" :goods_expense_cashbacked_amount="data.goods_expense_cashbacked_amount"></data>
                </div>
                <div class="dote"></div>
            </cell>
            <cell>
                <div v-if="isNull" class="null">
                    <image :src="nullUrl" resize="contain" class="nullimage"></image>
                    <text class="nulltext">暂无相关信息</text>
                </div>
            </cell>
            <cell class="listBox" v-if="!isNull">
                <text class="listTitle">配送业绩（共{{totalsheet}}单）</text>
                <deliverysheet v-for="items in deliverysheet" :deliveryman="items" @click.native="redirect(items.no)"></deliverysheet>
            </cell>
            <loading v-if="isLoading" class="loading" @loading="onloading" :display="loadinging ? 'show' : 'hide'">
                <text class="indicator-text">加载中 ...</text>
                <loading-indicator class="indicator"></loading-indicator>
            </loading>
        </list>

        <div ref="header" class="header">
            <div class="topBox" :style="{height:top+'px'}"></div>
            <title-bar :head="title" class="title"></title-bar>
            <search @search="search"></search>
        </div>
    </div>

</template>

<script>
    import titleBar from './components/titleBar.vue';
    import datas from './components/data.vue';
    import filtrate from './components/filtrate.vue';
    import deliverysheet from './components/deliverysheet.vue';
    import search from './components/search.vue';
    import self from './self.config';

    const stream = weex.requireModule("stream");
    const navigator = weex.requireModule("navigator");
    const modal = weex.requireModule("modal");
    const storage = weex.requireModule("storage");
    const dom = weex.requireModule("dom");

    // const url = "info.js?id=4&start_time=&end_time=&realname=小屋";
    const dataArr = weex.config.bundleUrl.split("?")[1].split("&");
    // const dataArr = url.split("?")[1].split("&");
    const id = dataArr[0].split("=")[1];
    const realname = decodeURI(dataArr[1].split("=")[1]);
    const start_time = dataArr[2].split("=")[1];
    const end_time = dataArr[3].split("=")[1];


    export default {
        name: "index",
        data() {
            return {
                title: '配送业绩',
                top:"40",
                nullUrl:self.imgUrl + "img_kong.png",
                totalsheet:0,
                data:{
                    freight_amount:0,
                    freight_cashbacked_amount:0,
                    goods_expense_amount:0,
                    goods_expense_cashbacked_amount:0
                },
                request:{
                    keyword:'',
                    page:1
                },
                deliverysheet: [],
                refreshings: false,
                isNull:false,
                isText:false,
                loadinging:false,
                isLoading:false,
                height:0,
                listTop:0
            }
        },
        created: function () {
            const _self = this;
            this.title = realname+"配送业绩";
            this.getdata(start_time,end_time,this.request.page,this.request.keyword,res=>{
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
            setTimeout(function () {
                dom.getComponentRect(_self.$refs.header,res=>{
                    _self.height = res.size.height;
                    _self.listTop = res.size.height;
                });
            },50);
        },
        methods: {
            //获取获取数据
            getdata(start_time,end_time,page,keyword,callback){
                weex.requireModule("nativeModule").getToken(e=>{
                    stream.fetch({
                        method: "get",
                        url: self.devUrl + "api/company/deliveryman/"+id+"/delivery_sheets?start_sign_time="+start_time+"&end_sign_time="+end_time+"&page="+page+"&keyword="+encodeURI(keyword),
                        type: "json",
                        headers: {
                            "Accept": "application/vnd.logistic.v3+json",
                            "Authorization": "bearer " + e.token
                        }
                    },res=>{
                        callback(res);
                    })
                })
            },
            //数据处理
            dataProcessing(res){
                const _self = this;
                const data = res.data;
                if(data.data.length == 0){
                    _self.isNull = true;
                    _self.isLoading = false;
                }else{
                    _self.isNull = false;
                    _self.deliverysheet = data.data;
                }
                this.totalsheet = data.meta.statistics.total_sheet;
                _self.data.freight_amount = data.meta.statistics.freight_amount;
                _self.data.freight_cashbacked_amount = data.meta.statistics.freight_cashbacked_amount;
                _self.data.goods_expense_amount = data.meta.statistics.goods_expense_amount;
                _self.data.goods_expense_cashbacked_amount  = data.meta.statistics.goods_expense_cashbacked_amount;
                if(_self.deliverysheet.length == 15){
                    _self.isLoading = true;
                }else{
                    _self.isLoading = false;
                }

            },
            //数据初始化
            dataReset(){
                this.data.freight_amount = 0;
                this.data.freight_cashbacked_amount = 0;
                this.data.goods_expense_amount = 0;
                this.data.goods_expense_cashbacked_amount  = 0;
            },
            //搜索数据
            search(value){
                const _self=this;
                this.request.keyword = value;
                this.getdata(start_time,end_time,this.request.page,this.request.keyword,res=>{
                    _self.dataProcessing(res);
                });
            },
            //上划加载
            onloading () {
                const _self = this;
                _self.loadinging = true;
                setTimeout(() => {
                    _self.loadinging = false;
                    _self.request.page++;
                    this.getdata(start_time,end_time,this.request.page,this.request.keyword,res=>{
                        const data = res.data;
                        if(data.data.length != 0){
                            for(let i in data.data){
                                _self.deliverysheet.push(data.data[i]);
                            }
                        }
                    })
                }, 500)
            },
            //下拉刷新
            onrefresh:function () {
                const _self = this;
                _self.refreshings = true;
                _self.request.page = 1;
                // this.getdata(start_time,end_time,this.request.page,this.request.keyword,res=>{
                //     _self.dataProcessing(res);
                // });
                setTimeout(function () {
                    _self.refreshings = false;
                }, 1000)
            },
            //跳转运单详情
            redirect(no){
                weex.requireModule("nativeModule").redirectOrder(JSON.stringify({
                    order_id:no
                }));
            }

        },
        components: {
            'title-bar': titleBar,
            'data': datas,
            'filtrate': filtrate,
            'deliverysheet': deliverysheet,
            'search':search
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
        background-color: #fff;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
        border-top-width: 1px;
        border-top-color: #dddddd;
    }
    .listTitle{
        width: 750px;
        height: 88px;
        line-height: 88px;
        padding-left: 30px;
        font-size: 28px;
        color: #666666;
        background-color: #f8f8f8;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
        border-top-width: 1px;
        border-top-color: #dddddd;
    }
    .null{
        width: 750px;
        align-items: center;
        position: absolute;
    }
    .nullimage{
        width: 214px;
        height: 254px;
    }
    .nulltext{
        font-size: 30px;
        color: #999;
    }
</style>