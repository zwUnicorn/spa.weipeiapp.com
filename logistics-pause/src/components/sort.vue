<template>
    <div class="wrapper">
        <text :class="[items.isActive?'isSort':'sort']" v-for="(items,index) in sort" @click="choice(index,items.sortStr)">{{items.text}}</text>
    </div>
</template>

<script>
    export default {
        name: "sort",
        props:["sort"],
        methods:{
            choice:function (index,str) {
                this.$emit("change",index,str);
            }
        }
    }
</script>

<style scoped>
    .wrapper{
        width: 750px;
        padding-left: 30px;
        padding-right: 30px;
    }
    .sort{
        font-size: 28px;
        color: #999999;
        height: 98px;
        line-height: 98px;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
    }
    .isSort{
        color: #3c98e6;
        height: 98px;
        font-size: 28px;
        line-height: 98px;
        border-bottom-width: 1px;
        border-bottom-color: #dddddd;
    }
</style>