<template>
    <div class="wrapper">
        <input class="input" type="text" placeholder="搜索" @change="search" value="">
    </div>
</template>

<script>
    const modal =weex.requireModule("modal");


    export default {
        name: "search",
        methods:{
            search:function (event) {
                console.log(event.value);
                this.$emit("search",event.value);
            }
        }
    }
</script>

<style scoped>
    .wrapper{
        align-items: center;
        background-color: #fff;
        padding-top: 15px;
        padding-bottom: 15px;
    }
    .input{
        width: 690px;
        height: 72px;
        padding-left: 30px;
        font-size: 28px;
        background-color: #efefef;
        placeholder-color:#999999;
        border-radius: 10px;
    }
</style>