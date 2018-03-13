<template>
    <div class="wrapper">
        <input class="input" type="text" placeholder="搜索" @change="serch" value="">
    </div>
</template>

<script>
    const modal =weex.requireModule("modal");


    export default {
        name: "search",
        methods:{
            serch:function (event) {
                this.$emit("serch",event.value);
            }
        }
    }
</script>

<style scoped>
    .wrapper{
        align-items: center;
    }
    .input{
        width: 690px;
        height: 72px;
        padding-left: 30px;
        font-size: 28px;
        background-color: #efefef;
        placeholder-color:#999999;
        border-radius: 4px;
    }
</style>