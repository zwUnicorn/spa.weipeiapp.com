<template>
    <div class="wrapper">
        <title-bar :head="title" :subhead="subtitle"></title-bar>
        <scroller class="scroller" show-scrollbar="false">
            <refresh class="refresh" @refresh="onRefresh" :display="refreshing ? 'show' : 'hide'">
                <image class="load" :src="loadUrl" resize="contain" ref="load"></image>
            </refresh>
            <div class="personInfo">
                <div class="info-total">
                    <text class="total-title font-color">当前可用额度</text>
                    <text class="total-num num-color">￥{{availableAmount}}</text>
                </div>
                <div class="info-type">
                    <div class="type">
                        <text class="type-name font-color">使用额度</text>
                        <text class="type-num num-color">￥{{usedAmount}}</text>
                    </div>
                    <div class="type">
                        <text class="type-name font-color">冻结额度</text>
                        <text class="type-num num-color">￥{{frozenAmount}}</text>
                    </div>
                </div>
            </div>
            <div class="query" @click="redirect ('history')">
                <text class="queryText">查询历史白条</text>
                <image class="listarrow" :src="listarrow" resize="contain"></image>
            </div>
            <div class="dote"></div>
            <div class="white-type">
                <div class="public type-title">
                    <text class="text">待出白条</text>
                    <text class="text">{{billing.length}}张</text>
                </div>
                <div class="white-type">
                    <info-bar :info="item" v-for="item in billing" @click.native="jump(item.order_no)"></info-bar>
                </div>
            </div>
            <div class="dote"></div>
            <div class="white-type">
                <div class="public type-title">
                    <text class="text">已出白条</text>
                    <text class="text">{{billed.length}}张</text>
                </div>
                <div class="white-type">
                    <info-bar :info="items" v-for="items in billed" @click.native="jump(items.order_no)"></info-bar>
                </div>
            </div>
        </scroller>
    </div>
</template>

<style scoped>
    .public{
        width: 718px;
        margin-left: 32px;
        padding-right: 32px;
    }
    .wrapper {
        width: 750px;
        align-items: center;
    }
    .scroller{
        position: fixed;
        width: 750px;
        top: 128px;
        bottom: 0;
    }
    .personInfo{
        width: 750px;
        height: 378px;
        background-image: linear-gradient(to bottom,#44b3e1,#3f91df);
    }
    .info-total{
        margin-left: 32px;
        margin-right: 32px;
        height: 216px;
        border-bottom-width: 1px;
        border-bottom-color: #67b2e6;
        align-items: center;
    }
    .refresh{
        background-color: #44b3e1;
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
    .listarrow{
        width: 20px;
        height: 20px;
        margin-left: 10px;
    }
    .font-color{
        color: #c4def4;
        margin-bottom: 18px;
    }
    .num-color{
        color: #ffffff;
    }
    .total-title{
        font-size: 26px;
        margin-top: 60px;
    }
    .total-num{
        font-size: 54px;
    }
    .info-type{
        margin-right: 32px;
        margin-left: 32px;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        height: 158px;
    }
    .type{
        flex: 1;
        align-items: center;
    }
    .type-name{
        font-size: 24px;
    }
    .type-num{
        font-size: 36px;
    }
    .query{
        width: 750px;
        height: 88px;
        flex-direction: row;
        justify-content: center;
        align-items: center;
    }
    .queryText{
        font-size: 28px;
        color: #44b3e1;
    }
    .dote{
        width: 750px;
        height: 24px;
        background-color: #f8f8f8;
        border-top-color: #dddddd;
        border-top-width: 1px;
        border-bottom-color: #dddddd;
        border-bottom-width: 1px;
    }
    .white-type{
        width: 750px;
    }
    .type-title{
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        height: 88px;
    }
    .text{
        font-size: 30px;
        color: #333333;
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

    const token = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOjE3MTY1LCJpc3MiOiJodHRwOlwvXC9hcGkuZGV2LndlaXBlaWFwcC5jb21cL3YzXC9vYXV0aFwvdG9rZW4iLCJpYXQiOjE0Njc4Nzg2ODksImV4cCI6MTUxOTIzOTg1OCwibmJmIjoxNTEyMDM5ODU4LCJqdGkiOiI4YzUwZjUxZGE5NzM4ZTA0MTc2YzIxMDQzYzU0ZDJmZiIsInJvbGUiOjEsImFjY291bnRzX2lkIjoxNjkyNX0.cw5jyLjOhPcNke-Ur2IKsS4Gyy3jgRrvqoseMkqP_Po'
    const getData = function (url,token,callback) {
        stream.fetch({
            method:'get',
            url: self.devUrl+'v3/'+url,
            type:'json',
            headers:{
                'Content-Type':'application/json',
                'accept':'application/vnd.api.v3+json',
                'authorization':'bearer '+ token
            }
        },callback)
    }
    export default {
        data(){
            return{
                title: '维配白条',
                subtitle:'',
                logoUrl: self.imgUrl+'img_open.png',
                listarrow:self.imgUrl + "btn_bluearrow.png",
                loadUrl:self.imgUrl + 'loading_white.png',
                availableAmount:'0',
                usedAmount:'0',
                frozenAmount:'0',
                code:'2',
                billing:[],
                billed:[],
                message:'1',
                refreshing:false,
                deg:720,
            }
        },
        mounted:function () {
            const _self = this;
           weex.requireModule('nativeModule').getToken(function callback(res) {
               storage.setItem("token",res.token);
                getData('white-finance/account',res.token,res=>{
                    if(res.data.status_code=='0'){
                        const data = res.data.data
                        _self.availableAmount = data.availableAmount
                        _self.usedAmount = data.usedAmount
                        _self.frozenAmount = data.frozenAmount
                    }
                })
                getData('white-finance?is_single=true&status=',res.token,res=>{
                    console.log(res)
                    const data = res.data.data;
                    for (let i=0;i<data.length;i++){
                        if(data[i].status =='billing'){
                            _self.billing.unshift(data[i])
                        }else if(data[i].status =='billed' || data[i].status =='paid_off' || data[i].status =='paying'){
                            _self.billed.unshift(data[i]);
                        }
                    }
                    for (let i=0;i<_self.billing.length;i++){
                        _self.billing[i].status = ' ';
                    }
                    for (let i=0;i<_self.billed.length;i++){
                        if(_self.billed[i].status == 'billed'){
                            _self.billed[i].status = '待还款';
                        }else{
                            _self.billed[i].status = '已还款';
                        }
                    }
                })
               getData('white-finance/unauthorized/administrator',res.token,res=>{
                   if(res.data.status_code == '0'){
                       _self.subtitle = '使用权限';
                   }
               })
           })
        },
        components:{
          'title-bar':titlebar,
            'info-bar':infobar
        },
        methods: {
            redirect:function (str) {
                navigator.push({
                    url: self.hostUrl+str+'.js'
                })
            },
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
                this.refreshing = true;
                setTimeout(() => {
                    this.refreshing = false
                }, 1000);
                
                storage.getItem("token",function (e) {
                    getData('white-finance/account',e.data,res=>{
                        if(res.data.status_code=='0'){
                            const data = res.data.data
                            _self.availableAmount = data.availableAmount
                            _self.usedAmount = data.usedAmount
                            _self.frozenAmount = data.frozenAmount
                        }
                    })
                    getData('white-finance?is_single=true&status=',e.data,res=>{
                        const data = res.data.data;
                        _self.billing = [];
                        _self.billed = [];
                        for (let i=0;i<data.length;i++){
                            if(data[i].status =='billing'){
                                _self.billing.unshift(data[i])
                            }else if(data[i].status =='billed' || data[i].status =='paid_off' || data[i].status =='paying'){
                                _self.billed.unshift(data[i]);
                            }
                        }
                        for (let i=0;i<_self.billing.length;i++){
                            _self.billing[i].status = ' ';
                        }
                        for (let i=0;i<_self.billed.length;i++){
                            if(_self.billed[i].status == 'billed'){
                                _self.billed[i].status = '待还款';
                            }else{
                                _self.billed[i].status = '已还款';
                            }
                        }
                    })
                })
            }
        }
    }
</script>