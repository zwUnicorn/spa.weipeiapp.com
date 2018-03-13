<template>
    <div class="wrapper">
        <title-bar :head="title" :subhead="subtitle"></title-bar>
        <div class="detailBox">
            <div class="detail">
                <text class="detailTitle">{{created_at}}&nbsp;白条金额&nbsp;￥{{amount}}</text>
                <div class="detailContent">
                    <text class="center detailName">{{name}}</text>
                    <text class="center detailNum">￥{{amount}}</text>
                    <text class="center detailTime" v-if="noBoolean">出款日期：{{created_at.substr(0,10)}}</text>
                    <text class="center pay" @click="pay" v-if="payBoolean">立即还款</text>
                    <text class="center detailText" v-if="waitBoolean">未出款，等待相应采购单签收后出款</text>
                    <text class="center detailTime" v-if="payBoolean">本期还款截止日期：{{paid_off_at.substr(0,10)}}</text>
                    <text class="center detailQuery" @click="redirect">查看相关采购单&nbsp;></text>
                </div>
                <div class="dot dot-left"></div>
                <div class="dot dot-right"></div>
            </div>
            <div class="payrecord" v-if="listBoolean">
                <div class="paytitle">
                    <text class="paySize payColorOne">还款记录</text>
                    <text class="paySize payColorTwo">共1期</text>
                </div>
                <div class="payinfo">
                    <div class="info">
                        <text class="paySize payColorTwo payDate">第1期</text>
                        <text class="paySize payColorOne payNum">￥{{amount}}</text>
                        <text class="paySize payColorThree payState">{{payStatus}}</text>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
    .wrapper{
        align-items: center;
    }
    .center{
        text-align: center;
    }
    .detailBox{
        width: 750px;
        align-items: center;
        position: fixed;
        top: 128px;
        bottom: 0;
        background-color: #44b3e1;
    }
    .detail{
        width: 686px;
        margin-top: 32px;
        position: relative;
    }
    .detailTitle{
        font-size: 26px;
        color: #333333;
        height: 88px;
        background-color: #eef3f5;
        border-radius: 6px;
        text-align: center;
        line-height: 88px;
    }
    .detailContent{
        background-color: #ffffff;
    }
    .detailName{
        margin-top: 64px;
        font-size: 28px;
        color: #999999;
        padding-left: 36px;
        padding-right: 36px;
        text-align: center;
        line-height: 42px;
    }
    .detailNum{
        font-size: 54px;
        color: #f5a623;
        margin-top: 32px;
    }
    .pay{
        width: 560px;
        height: 80px;
        text-align: center;
        line-height: 80px;
        background-image: linear-gradient(to bottom,#44b3e1,#449de1);
        border-radius: 10px;
        font-size: 30px;
        color: #fff;
        margin-top: 64px;
        margin-left: 63px;
        margin-right: 63px;
    }
    .detailTime{
        font-size: 26px;
        color: #666666;
        margin-top: 24px;
    }
    .detailText{
        font-size: 30px;
        color: #666666;
        margin-top: 60px;
    }
    .detailQuery{
        font-size: 30px;
        color: #44b3e1;
        margin-top: 32px;
        margin-bottom: 64px;
    }
    .dot{
        width: 20px;
        height: 20px;
        background-color: #44b3e1;
        border-radius: 20px;
        position: absolute;
        top: 78px;
    }
    .dot-left{
        left: -10px;
    }
    .dot-right{
        right: -10px;
    }
    .paySize{
        font-size: 28px;
    }
    .payColorOne{
        color: #333333;
    }
    .payColorTwo{
        color: #666666;
    }
    .payColorThree{
        color: #44b3e1;
    }
    .payrecord{
        width: 686px;
        margin-top: 24px;
    }
    .paytitle{
        width: 686px;
        height: 88px;
        padding-left: 24px;
        padding-right: 24px;
        background-color: #eef3f5;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
    }
    .info{
        flex-direction: row;
        height: 98px;
        background-color: #fff;
        padding-left: 24px;
        padding-right: 24px;
        align-items: center;
    }
    .payDate{
        flex: 2;
    }
    .payNum{
        flex: 5;
    }
    .payState{
        flex: 3;
        text-align: right;
    }
</style>

<script>
    import self from './self.config'
    import titlebar from './components/titlebar.vue'

    const stream = weex.requireModule('stream');
    const storage = weex.requireModule('storage');

    export default {
        data(){
            return {
                title: '待出款白条',
                subtitle: '',
                logoUrl: self.imgUrl + 'img_open.png',
                payBoolean:false,
                noBoolean:false,
                listBoolean:false,
                waitBoolean:false,
                payStatus:"未还",
                created_at:'0',
                amount:'0',
                name:'0',
                paid_off_at:'0',
                order_id:0,
                order_no:0
            }
        },
        mounted:function () {
            const _self =this;
            const orderId = weex.config.bundleUrl.split("id=")[1];
            storage.getItem("token",function (e) {
                stream.fetch({
                    method:'get',
                    url: self.devUrl+'v3/white-finance/'+orderId,
                    type:'json',
                    headers:{
                        'Content-Type':'application/x-www-form-urlencoded',
                        'accept':'application/vnd.api.v3+json',
                        'authorization':'bearer '+ e.data
                    }
                },res=>{
                    _self.order_id = res.data.data.order_id;
                    _self.order_no = res.data.data.order_no;
                    if(res.data.status_code == '0'){
                        const data = res.data.data;
                        _self.created_at = data.created_at;
                        _self.amount = data.amount_to_fen;
                        _self.name = data.title;
                        _self.paid_off_at = data.paid_off_at;
                        if(data.status == 'billing'){
                            waitBoolean = true;
                        }else if(data.status == 'billed'){
                            _self.title = "待还款白条";
                            _self.payBoolean =true;
                            _self.noBoolean = true;
                            _self.listBoolean = true;

                        }else if(data.status == 'paid_off' || data.status == 'paying'){
                            _self.title = "已还款白条";
                            _self.noBoolean =true;
                            _self.listBoolean=true;
                            _self.payStatus = data.paid_off_at.substr(0,10);
                        }
                    }
                })
            })
        },
        components:{
          'title-bar':titlebar,
        },
        methods: {
            pay:function () {
                const _self= this;
                weex.requireModule("nativeModule").repayment(JSON.stringify({
                    amount:_self.amount,
                    order_no:_self.order_no,
                }),function callback(res) {
                    if(res.status == '1'){
                        _self.title = "已还款白条";
                        _self.payBoolean = false;
                        _self.payStatus = _self.paid_off_at.substr(0,10);
                    }
                })
            },
            redirect:function () {
                weex.requireModule("nativeModule").redirectOrder(JSON.stringify({
                    order_id:this.order_id
                }),function callback() {
                    console.log(1)
                })
            }
        }
    }
</script>