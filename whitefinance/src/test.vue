<template>
    <div class="text">
        <text @click="click">点击我测试</text>
        <text>{{str}}</text>
    </div>
</template>
<style>
    .text{
        margin-top: 500px;
    }
</style>
<script>

    function s(d, a, b) {
        console.log(d);
    }

    s(222, 222, 333 ,4444);

    module.exports = {
        data(){
            return{
                str:'aa'
            }
        },
        methods: {
            click: function() {
                console.log(1);
                const _self = this;
                weex.requireModule('myModule').printLog('ddddd',   function fun(param, ) {
                    if (param == 'test') {
                        _self.str='ddddddddd';
                    }
                })
            }
        },





    }
</script>
