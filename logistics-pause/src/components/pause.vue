<template>
    <div class="wrapper">
        <div class="stationList">
            <text v-for="(item,index) in pause" :class="[item.isActive?'isPause' : 'pause']" @click="choice(index)">{{item.value}}</text>
        </div>
        <div class="btngroup">
            <text class="no" @click="choiceAll">取消</text>
            <text class="confirm" @click="getData">完成</text>
        </div>
    </div>
</template>

<script>
    import self from "../self.config";
    const stream = weex.requireModule("stream");
    const modal = weex.requireModule("modal");
    const storage = weex.requireModule("storage");
    export default {
        name: "station",
        props:["pause"],
        data(){
            return{
                all:true,
            }
        },
        methods:{
            choice:function (index) {
                for (let i in this.pause){
                    if(i != index){
                        this.pause[i].isActive = false;
                    }
                }
                this.pause[index].isActive = !this.pause[index].isActive;
            },
            choiceAll:function () {
               this.$emit("cancel");
            },
            getData:function () {
                let ids;
                let value;
                for (let i in this.pause){
                    if(this.pause[i].isActive){
                        ids = this.pause[i].id;
                        value = this.pause[i].value;
                    }
                }
                if(ids == null){
                    this.$emit("getId",-1,'all');
                }else{
                    this.$emit("getId",ids,value);
                }

            }
        }
    }
</script>

<style scoped>
    .wrapper{
        width: 750px;
    }
    .pause{
        height: 72px;
        line-height: 72px;
        padding-left: 40px;
        padding-right: 40px;
        margin-right: 30px;
        margin-bottom: 30px;
        color: #333333;
        font-size: 28px;
        border-color: #dddddd;
        border-width: 2px;
    }
    .isPause{
        height: 72px;
        line-height: 72px;
        padding-left: 40px;
        padding-right: 40px;
        margin-right: 30px;
        margin-bottom: 30px;
        color: #3c98e6;
        font-size: 28px;
        border-color: #3c98e6;
        background-color: #e4f3f9;
        border-width: 2px;
    }
    .stationList{
        padding-left: 30px;
        padding-right: 30px;
        padding-bottom: 30px;
        padding-top: 30px;
        flex-direction: row;
        flex-wrap: wrap;
    }
    .btngroup{
        flex-direction: row;
        border-top-width: 1px;
        border-top-color: #dddddd;
        width: 750px;
    }
    .no{
        font-size: 30px;
        color: #3c98e6;
        flex: 1;
        text-align: center;
        height: 88px;
        line-height: 88px;
    }
    .confirm{
        font-size: 30px;
        color: #fff;
        background-color: #3c98e6;
        text-align: center;
        height: 88px;
        line-height: 88px;
        flex: 1;
    }
</style>