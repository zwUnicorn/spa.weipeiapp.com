<template>
    <div class="wrapper">
        <title-bar :head="title" :subhead="subtitle" class="title"></title-bar>
        <scroller class="scroller" show-scrollbar="false">
            <div class="config">
                <div class="configBox">
                    <text class="configTitle">管理员</text>
                    <text class="adminName">{{admin}}</text>
                </div>
                <div class="configBox">
                    <text class="configTitle">白条权限</text>
                    <div class="adminList">
                        <div class="adminInfo" v-for="(item,index) in user">
                            <text class="infoName">{{item.repair_shop_account_name}}</text>
                            <div class="switchBox">
                                <text class="infoState" v-if="item.is_authorized">授权</text>
                                <text class="infoState" v-if="!item.is_authorized">未授权</text>
                                <switch class="switch" :checked="item.is_authorized" @change="change(index,item.repair_shop_account_id)"></switch>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </scroller>
    </div>
</template>

<style scoped>
    .scroller{
        position: fixed;
        width: 750px;
        top: 128px;
        bottom: 0;
        background-color: #f8f8f8;
    }
    .title{
        position: fixed;
        top: 0;
    }
    .configBox{
        width: 750px;
        border-bottom-width: 2px;
        border-bottom-color: #dddddd;
    }
    .configTitle{
        font-size: 26px;
        color: #999999;
        padding-top: 32px;
        padding-bottom: 16px;
        padding-left: 32px;
        padding-right: 32px;
    }
    .adminName{
        height: 88px;
        font-size: 28px;
        color: #333333;
        padding-left: 32px;
        padding-right: 32px;
        background-color: #fff;
        line-height: 88px;
        border-top-width: 2px;
        border-top-color: #dddddd;
    }
    .adminList{
        background-color: #fff;
        border-top-width: 2px;
        border-top-color: #dddddd;
    }
    .adminInfo{
        margin-left: 32px;
        padding-right: 32px;
        height: 88px;
        align-items: center;
        flex-direction: row;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
        justify-content: space-between;
        margin-bottom: -1px;
    }
    .infoName{
        font-size: 28px;
        color: #333333;
    }
    .infoState{
        font-size: 26px;
        color: #999999;
        margin-right: 15px;
    }
    .switchBox{
        flex-direction: row;
        align-items: center;
    }
    .switch{
        align-items: flex-start;
    }
</style>

<script>
    import self from './self.config'
    import titlebar from './components/titlebar.vue'

    const stream = weex.requireModule('stream');
    const storage = weex.requireModule('storage');

    const changeStatus = function (str,list) {
        for(let i=0;i<list.length;i++){
            if(list[i].is_administrator == true){
                str.admin = list[i].repair_shop_account_name;
            }
        }
    }
    const changeLimit = function (id,url,token) {
        stream.fetch({
            method:'post',
            url: self.devUrl+'v3/white-finance/'+id+'/'+url,
            type:'json',
            headers:{
                'Content-Type':'application/x-www-form-urlencoded',
                'accept':'application/vnd.api.v3+json',
                'authorization':'bearer '+ token
            }
        })
    }
    export default {
        data(){
            return{
                title: '使用权限',
                subtitle:'',
                logoUrl: self.imgUrl+'img_open.png',
                user:[],
                admin:''
            }
        },
        created:function () {
            const _self = this;
            storage.getItem("token",function (e) {
                stream.fetch({
                    method:'get',
                    url: self.devUrl+'v3/white-finance/unauthorized/users',
                    type:'json',
                    headers:{
                        'Content-Type':'application/x-www-form-urlencoded',
                        'accept':'application/vnd.api.v3+json',
                        'authorization':'bearer '+ e.data
                    }
                },res=>{
                    if(res.data.status_code == 0){
                        const data = res.data.data;
                        _self.user = data;
                        changeStatus(_self,data);
                    }
                })
            })
        },
        components:{
          'title-bar':titlebar
        },
        methods: {
            change: function (index,repair_shop_account_id) {
                const _self = this;
                _self.user[index].is_authorized = !_self.user[index].is_authorized;
                storage.getItem("token",function (e) {
                    if(_self.user[index].is_authorized){
                        changeLimit(repair_shop_account_id,'authorization',e.data);
                    }else{
                        changeLimit(repair_shop_account_id,'unauthorization',e.data);
                    }

                })
            }
        }
    }
</script>