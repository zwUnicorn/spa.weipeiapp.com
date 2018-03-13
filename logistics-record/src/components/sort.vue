<template>
    <div class="wrapper">
        <div class="time" @click="choiceTime(0)">
            <text class="font">起始日期：{{start_time}}</text>
            <image :src="arrowDown" class="image" resize="contain"></image>
        </div>
        <div class="time" style="border-bottom-width: 0" @click="choiceTime(1)">
            <text class="font">结束日期：{{end_time}}</text>
            <image :src="arrowDown" class="image" resize="contain"></image>
        </div>
        <div class="btns">
            <text class="btn reset" @click="reset">重置</text>
            <text class="btn sure" @click="timeChoose">确定</text>
        </div>
    </div>
</template>

<script>
    import self from '../self.config';

    const picker = weex.requireModule("picker");
    const modal = weex.requireModule("modal");
    // const platform = weex.config.env.platform;
    export default {
        name: "sort",
        props:["sort"],
        data(){
            return{
                arrowDown:self.imgUrl+'icon_arrow_down.png',
                start_time:'',
                end_time:''
            }
        },
        methods:{
            //获取当前时间
            getdate(){
                var date = new Date();
                var seperator1 = "-";
                var year = date.getFullYear();
                var month = date.getMonth() + 1;
                var strDate = date.getDate();
                if (month >= 1 && month <= 9) {
                    month = "0" + month;
                }
                if (strDate >= 0 && strDate <= 9) {
                    strDate = "0" + strDate;
                }
                var currentdate = year + seperator1 + month + seperator1 + strDate;
                return currentdate;
            },
            //调去原生date选择
            setdate(type,mindate,maxdate){
                const _self =this;
                weex.requireModule("nativeModule").getDate(JSON.stringify({
                    mindate:mindate,
                    maxdate:maxdate
                }),function callback(res) {
                    if(type == 0){
                        _self.start_time = res.date;
                    }else {
                        _self.end_time = res.date;
                    }
                })

            },
            choice:function (index,str) {
                this.$emit("change",index,str);
            },
            choiceTime(type){
                let mindate='';
                let maxdate='';
                if(type == 0){
                    if(this.end_time == ""){
                        maxdate = this.getdate();
                    }else{
                        maxdate = this.end_time;
                    }
                    this.setdate(type,mindate,maxdate);
                }else{
                    maxdate = this.getdate();
                    if(this.start_time != ""){
                        mindate = this.start_time;
                    }
                    this.setdate(type,mindate,maxdate);
                }

            },
            timeChoose(){
                this.$emit("timechoose",this.start_time,this.end_time);
            },
            reset(){
                this.start_time = "";
                this.end_time = "";
                this.timeChoose();
            }
        }
    }
</script>

<style scoped>
    .wrapper{
        width: 750px;
        align-items: center;
    }
    .time{
        width: 690px;
        height: 88px;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        border-bottom-color: #dddddd;
        border-bottom-width: 1px;
    }
    .font{
        font-size: 30px;
        color: #666666;
    }
    .image{
        width: 18px;
        height: 18px;
    }
    .btns{
        flex-direction: row;
        height: 88px;
        width: 750px;
    }
    .btn{
        flex: 1;
        font-size: 30px;
        color: #3c98e6;
        text-align: center;
        line-height: 88px;
        border-top-color: #dddddd;
        border-top-width: 1px;
    }
    .sure{
        background-color: #3c98e6;
        color: #ffffff;
        border-top-color: #3c98e6;
    }
</style>