<template>
    <div class="wrapper">
        <div class="mask"></div>
        <div class="list">
            <sort></sort>
        </div>
    </div>
</template>

<script>
    import sort from './sort.vue'
    export default {
        name: "dropdown",
        components:{
            "sort":sort
        }
    }
</script>

<style scoped>
    .wrapper{
        width: 750px;
    }
    .list{
        align-items: center;
        background-color: #fff;
    }
    .mask{
        position: absolute;
        bottom: 0;
        top:0;
        left: 0;
        right: 0;
        background: black;
        opacity: .3;
    }
</style>