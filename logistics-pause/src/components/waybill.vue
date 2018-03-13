<template>
    <div class="wrapper">
        <div class="listTitle" @click="choiceAll">
            <image class="image" :src="allchoice ? choiceUrl : nochoiceUrl" resize="contain"></image>
            <text class="title"  >欠款运单全选</text>
        </div>
        <div class="box" v-for="(items,index) in waybill" @click.self="choice(index)">
            <image class="image" :src="items.isActive ? choiceUrl : nochoiceUrl" resize="contain"></image>
            <div class="waybill">
                <div class="name row">
                    <text v-if="items.appointment_sheet_id != ' '" class="fontMine bgYellow status" style="margin-right: 10px">预</text>
                    <text class="fontLarge colorBlack" @click="redirectOrder(items.no)">{{items.goodses_no}}</text>
                    <text v-if="items.is_freight_changed || items.is_goods_expense_changed" class="fontMine bgRed status">改</text>
                    <text v-if="items.is_lose_reported" class="fontMine bgOrange status">挂失</text>
                    <text v-if="items.is_settled" class="fontMine bgBlue status">结</text>
                </div>
                <div class="row between">
                    <div class="row bottom">
                        <text class="fontMidlle colorGary">到付&nbsp;￥{{moneyData[index].reached_receivable_freight}}</text>
                        <text class="fontMidlle colorGary">货款&nbsp;￥{{moneyData[index].receivable_goods_expense}}</text>
                    </div>
                    <text class="fontLarge color">￥{{(moneyData[index].reached_receivable_freight + moneyData[index].receivable_goods_expense).toFixed(2)}}</text>
                </div>
                <text class="fontMidlle colorGary bottom">{{items.sender+"—"+items.receiver}}</text>
                <text v-if="items.cant_settle_reason != ''" class="fontMidlle colorGary bottom">原因：{{items.pauseReasons.data[0].reason}}</text>
                <text v-if="items.pause_category_str !=''" class="fontMidlle colorMine bottom">分类：{{items.pause_category_str}}</text>
                <text class="fontSmall colorMine">{{items.pauseReasons.data[0].created_at}}</text>
            </div>
        </div>
    </div>
</template>

<script>
    import self from '../self.config';

    export default {
        name: "waybill",
        props:['waybill',"allchoice",'moneyData'],
        data(){
            return{
                nochoiceUrl: self.imgUrl+'icon_choice_normal.png',
                choiceUrl: self.imgUrl+'icon_choice_hover.png'
            }
        },
        methods:{
            choice:function (index) {
                this.$emit("change",index,!this.allchoice);
            },
            choiceAll:function () {
                this.$emit("changeAll");
            },
            redirectOrder:function (no) {
                weex.requireModule("nativeModule").redirectOrder(JSON.stringify({
                    order_id:no
                }));
            }

        }
    }
</script>

<style scoped>
    .wrapper{
        align-items: flex-end;
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
</style>