<template>
    <div class="wrapper">
        <div class="stationList">
            <text v-for="(item,index) in row" :class="[item.isActive ? 'stationActive':'station']" @click="choice(index)" style="margin-top: 30px">{{item.name}}</text>
        </div>
        <div class="btngroup">
            <text class="all" @click="choiceAll">{{all?"全选":"取消全选"}}</text>
            <text class="confirm" @click="get_station">完成</text>
        </div>
    </div>
</template>

<script>
    import self from '../self.config';
    const modal = weex.requireModule("modal");
    export default {
        name: "station",
        props:["row"],
        data(){
          return{
              all: true
          }
        },
        methods:{
            choice:function (i) {
                this.row[i].isActive = !this.row[i].isActive;
                let no=0;
                for(let i in this.row){
                    if(this.row[i].isActive){
                        no++;
                    }
                }
                if(no==this.row.length){
                    this.all=false;
                }else {
                    this.all=true;
                }
            },
            choiceAll:function () {
                if(this.all){
                    for (let i in this.row){
                        this.row[i].isActive = true;
                    }
                }else {
                    for (let i in this.row){
                        this.row[i].isActive = false;
                    }
                }
                this.all = !this.all;
            },
            get_station:function () {
                let stations=[];
                for (let i in this.row){
                    if(this.row[i].isActive){
                        stations.push(this.row[i].station_id)
                    }
                }
                this.$emit("get_station",stations);
            }
        }
    }
</script>

<style scoped>
    .wrapper{
        width: 750px;
        padding-bottom: 90px;
    }
    .station{
        width: 210px;
        height: 72px;
        line-height: 72px;
        text-align: center;
        color: #333333;
        font-size: 28px;
        border-color: #dddddd;
        border-width: 2px;
        margin-left: 30px;
        margin-top: 30px;
    }
    .stationActive{
        width: 210px;
        height: 72px;
        line-height: 72px;
        text-align: center;
        font-size: 28px;
        border-color: #3c98e6;
        color: #3c98e6;
        background-color: #e4f3f9;
        border-width: 2px;
        margin-left: 30px;
        margin-top: 30px;
    }
    .stationList{
        padding-right: 30px;
        padding-bottom: 30px;
        flex-direction: row;
        flex-wrap: wrap;
    }
    /*.rowList{*/
        /*flex-direction: row;*/
        /*justify-content: space-between;*/
        /*width: 690px;*/
        /*margin-top: 30px;*/
    /*}*/
    .btngroup{
        flex-direction: row;
        border-top-width: 1px;
        border-top-color: #dddddd;
        width: 750px;
        position: absolute;
        bottom: 0;
    }
    .all{
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