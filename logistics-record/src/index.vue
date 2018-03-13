<template>
    <div class="wrapper">
        <list class="list" :style="{top:listTop+'px'}" ref="list">
            <refresh class="refresh" @refresh="onrefresh" :display="refreshing ? 'show' : 'hide'">
                <loading-indicator class="indicator"></loading-indicator>
            </refresh>
            <cell>
                <div class="dataBox">
                    <data :title="text" :freight_amount="datas.freight_amount" :goods_expense_amount="datas.goods_expense_amount" :freight_cashbacked_amount="datas.freight_cashbacked_amount" :goods_expense_cashbacked_amount="datas.goods_expense_cashbacked_amount"></data>
                </div>
                <div class="dote"></div>
            </cell>

            <cell class="listBox" v-if="!isNull">
                <text class="listTitle">配送业绩（共{{totalsheet}}单）</text>
                <deliveryman :deliveryman="items" v-for="items in deliverymen" @click.native="redirect(items.user_id,items.realname)"></deliveryman>
            </cell>
            <cell>
                <div v-if="isNull" class="null">
                    <image :src="nullUrl" resize="contain" class="nullimage"></image>
                    <text class="nulltext">暂无相关信息</text>
                </div>
            </cell>
        </list>
        <filtrate class="filtrate" :station_s="station_s"  @fresh="fresh" @get_station="get_station" @timechoose="timechoose" :style="{top:height+'px'}"></filtrate>
        <div ref="header" class="header">
            <div class="topBox" :style="{height:top+'px'}"></div>
            <title-bar :head="title" class="title"></title-bar>
        </div>
    </div>

</template>

<script>
    import titleBar from './components/titleBar.vue';
    import datas from './components/data.vue';
    import filtrate from './components/filtrate.vue';
    import deliveryman from './components/deliveryman.vue';
    import self from './self.config';

    const stream = weex.requireModule("stream");
    const navigator = weex.requireModule("navigator");
    const modal = weex.requireModule("modal");
    const storage = weex.requireModule("storage");
    const dom = weex.requireModule("dom");

    export default {
        name: "index",
        data() {
            return {
                title: '配送业绩',
                top:"40",
                station_s: '选择配送员',
                nullUrl:self.imgUrl + "img_kong.png",
                totalsheet:0,
                datas:{
                    freight_amount:0,
                    freight_cashbacked_amount:0,
                    goods_expense_amount:0,
                    goods_expense_cashbacked_amount:0
                },
                request:{
                    id:'',
                    start_time:'',
                    end_time:'',
                },
                deliverymen: [],
                refreshing: false,
                isNull:false,
                isText:false,
                showLoading:"hide",
                loadinging:false,
                isLoading:false,
                height:0,
                listTop:0
            }
        },
        created: function () {
            const _self = this;
            this.getdata(this.request.id,this.request.start_time,this.request.end_time);
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
                    if(res.size.height==0){
                        _self.height = 88;
                    }else{
                        _self.height = res.size.height;
                    }
                    _self.listTop = _self.height + 80;
                });
            },50);
        },
        methods: {
            //获取获取数据
            getdata(id,start_time,end_time){
                const _self = this;
                weex.requireModule("nativeModule").getToken(e=>{
                    stream.fetch({
                        method: "get",
                        url: self.devUrl + "api/company/deliveryman/achievement?"+id+"start_sign_time="+start_time+"&end_sign_time="+end_time,
                        type: "json",
                        headers: {
                            "Accept": "application/vnd.logistic.v3+json",
                            "Authorization": "bearer " + e.token
                        }
                    },res=>{
                        _self.dataProcessing(res);
                    })
                })
            },
            //数据处理
            dataProcessing(res){
                const _self=this;
                if(res.data.data.length == 0){
                    _self.isNull = true;
                    _self.isLoading = false;
                    this.dataReset();
                }else{
                    _self.isNull = false;
                    this.dataReset();
                    this.deliverymen = res.data.data;
                    for (let i=0;i<this.deliverymen.length;i++){
                        this.datas.freight_amount += this.deliverymen[i].freight_amount;
                        this.datas.freight_cashbacked_amount += this.deliverymen[i].freight_cashbacked_amount;
                        this.datas.goods_expense_amount += this.deliverymen[i].goods_expense_amount;
                        this.datas.goods_expense_cashbacked_amount += this.deliverymen[i].goods_expense_cashbacked_amount;
                        this.totalsheet += this.deliverymen[i].total_sheet;
                    }
                }
            },
            //数据初始化
            dataReset(){
                this.totalsheet = 0;
                this.datas.freight_amount = 0;
                this.datas.freight_cashbacked_amount = 0;
                this.datas.goods_expense_amount = 0;
                this.datas.goods_expense_cashbacked_amount = 0;
            },
            //站点筛选
            get_station: function (stations) {
                const _self = this;
                _self.request.id='';
                _self.page = 1;
                for(let i in stations){
                    _self.request.id+=("user_id[]="+stations[i]+"&");
                }
                this.getdata(this.request.id,this.request.start_time,this.request.end_time);
            },
            //时间筛选
            timechoose(startTime,endTime){
                this.request.start_time = startTime;
                this.request.end_time = endTime;
                this.getdata(this.request.id,this.request.start_time,this.request.end_time);
            },
            //下拉刷新
            onrefresh:function () {
                const _self = this;
                _self.refreshing = true;
                _self.page = 1;
                this.getdata(this.request.id,this.request.start_time,this.request.end_time);
                setTimeout(() => {
                    _self.refreshing = false
                }, 1000)
            },
            //跳转商户运单列表
            redirect:function (id,name) {
                navigator.push({
                    url: self.hostUrl + "tets.js?id="+id+"&realname="+encodeURI(name)+"&start_time="+this.request.start_time+"&end_time="+this.request.end_time
                })
            }
        },
        components: {
            'title-bar': titleBar,
            'data': datas,
            'filtrate': filtrate,
            'deliveryman': deliveryman,
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
        justify-content: center;
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
        top: 100px;
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