<template>
    <div class="wrapper">
        <div class="listTitle" @click="choiceAll">
            <image class="image" :src="!isClick? cannotUrl : allchoice ? choiceUrl : nochoiceUrl" resize="contain"></image>
            <text class="title"  >月结运单</text>
        </div>
        <div class="box" v-for="(items,index) in waybill" @click="choice(index,items)">
            <image class="image" :src="items.can_cashback_amount == 0 ? cannotUrl : items.isActive ? choiceUrl : nochoiceUrl" resize="contain"></image>
            <div class="waybill">
                <div class="name row">
                    <text v-if="items.appointment_sheet_id != '0'" class="fontMine bgYellow status" style="margin-right: 10px">预</text>
                    <text class="fontLarge colorBlack" @click="redirectOrder(items.no)">{{items.goodses_no}}</text>
                    <text v-if="items.is_freight_changed || items.is_goods_expense_changed" class="fontMine bgRed status">改</text>
                    <text v-if="items.is_lose_reported" class="fontMine bgOrange status">挂失</text>
                    <text v-if="items.is_settled" class="fontMine bgBlue status">结</text>
                </div>
                <div class="row bottom">
                    <text v-if="items.unreach_receivable_freight != 0" class="fontMidlle colorGary right">现付￥{{items.unreach_receivable_freight.toFixed(2)}}</text>
                    <text v-if="items.back_freight != 0" class="fontMidlle colorGary right">补缴￥{{items.back_freight.toFixed(2)}}</text>
                    <text v-if="items.can_cashback_amount != 0" class="fontMidlle colorGary">(可回款￥{{items.can_cashback_amount}})</text>
                </div>
                <div class="row between" style="align-items: flex-start;">
                    <text class="fontMidlle colorGary bottom" style="flex: 4">{{items.sender}}—{{items.receiver}}</text>
                    <text class="fontLarge color" style="flex: 1;text-align: right">￥{{(items.unreach_receivable_freight + items.back_freight).toFixed(2)}}</text>
                </div>
                <text class="fontSmall colorMine">{{items.created_at}}</text>
                <text class="fontSmall colorMine">{{items.part_cant_cashback_reason}}</text>
            </div>
        </div>
    </div>
</template>

<script>
    import self from '../self.config';

    const modal = weex.requireModule("modal");

    export default {
        name: "waybill",
        props:['waybill',"allchoice","isClick"],
        data(){
            return{
                nochoiceUrl: self.imgUrl+'icon_choice_normal.png',
                choiceUrl: self.imgUrl+'icon_choice_hover.png',
                cannotUrl :self.imgUrl + 'icon_Prohibition.png',
            }
        },
        methods:{
            choice:function (index,data) {
                if(data.can_create_cashback_sheet){
                    this.$emit("change",index,!this.allchoice);
                }
            },
            choiceAll:function () {
                this.$emit("changeAll");
            },
            redirectOrder:function (no) {
                weex.requireModule("nativeModule").redirectOrder(JSON.stringify({
                    order_id:no
                }));
            },
            //获取不能回款提示信息
            getMsg(data){
                const un_status = data.unreach_freight_sheet_status;//现付创建回款单状态
                const back_status = data.back_freight_sheet_status;//补缴创建回款单状态
                const un = data.unreach_receivable_freight;//现付金额
                const back = data.back_freight;//补缴金额
                const backed_sign = data.backed_sign_time;//返单签收状态

                if(un != 0 && back == 0){
                    if(un_status != 0){
                        return "现付已创建回款单";
                    }
                }
                if(un == 0 && back != 0){
                    if(backed_sign != null){
                        return "补缴已创建回款单";
                    }else{
                        return "补缴未签收";
                    }
                }
                if(un != 0 && back != 0){
                    if(un_status){
                        if(backed_sign == null){
                            return "现付已创建回款单，补缴未签收"
                        }else{
                            return "现付已创建回款单"
                        }
                    }
                    if(back_status != 0){
                        if(un_status != 0){
                            return "现付、补缴已创建回款单";
                        }else{
                            return "补缴已创建回款单"
                        }

                    }
                }
            }

        }
    }
</script>

<style scoped>
    .wrapper{
        align-items: flex-end;
        background-color: #fff;
    }
    .box{
        width: 720px;
        padding-right: 30px;
        padding-bottom: 20px;
        padding-top: 20px;
        flex-direction: row;
        align-items: flex-start;
        border-bottom-color: #dddddd;
        border-bottom-width: 1px;
        margin-bottom: -1px;
    }
    .image{
        width: 40px;
        height: 40px;
        top: 1px;
    }
    .waybill{
        margin-left: 24px;
        width: 626px;
    }
    .row{
        flex-direction: row;
        align-items: center;
    }
    .fontLarge{
        font-size: 32px;
    }
    .fontMidlle{
        font-size: 26px;
    }
    .fontSmall{
        font-size: 24px;
    }
    .fontMine{
        font-size: 22px;
    }
    .colorGary{
        color: #666666;
    }
    .colorBlack{
        color: #333333;
    }
    .colorMine{
        color: #999999;
    }
    .color{
        color: #feae49;
    }
    .iscolor{
        color: blue;
    }
    .bgRed{
        background-color: #ee3f3f;
        margin-left: 10px;
    }
    .bgBlue{
        background-color: #6fafe4;
    }
    .bgOrange{
        background-color: #ff8213;
    }
    .bgYellow{
        background-color: #f6af23;
    }
    .status{
        color: #fff;
        border-radius: 4px;
        height: 36px;
        line-height: 36px;
        padding-right: 7px;
        padding-left: 7px;
        margin-left: 10px;
    }
    .between{
        justify-content: space-between;
    }
    .listTitle{
        flex-direction: row;
        align-items: center;
        width: 750px;
        height: 88px;
        background-color: #f8f8f8;
        padding-left: 30px;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
    }
    .title{
        font-size: 28px;
        margin-left: 24px;
        color: #666666;
    }
    .name{
        margin-bottom: 10px;
    }
    .bottom{
        margin-bottom: 5px;
    }
    .right{
        margin-right: 10px;
    }
</style>