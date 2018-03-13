<template>
    <div class="select-container">
        <div class="content">
            <content></content>
        </div>
        <div class="mask" id="mask" onclick="switchView"></div>
        <scroller class="scroller" id="scroller">
            <div class="options" id="options" style="top:{{top}};">
                <div repeat="{{status}}" class="cell" vid="{{id}}" onclick="onItemClick">
                    <text class="name {{id==statusId?'current': ''}}" if="{{id!=statusId}}">{{name}}</text>
                    <text class="name {{id==statusId?'current': ''}}" style="color: cornflowerblue;" if="{{id==statusId}}">{{name}}</text>
                    <image style="width: 32px;height: 32px;" src="{{flagSrc}}" if="{{id==statusId}}"></image>
                </div>
            </div>
            <div style="height: 90px"></div>
        </scroller>
        <div class="select" onclick="switchView">
            <text class="current-text">{{statusName}}</text>
            <image id="arrow" style="width: 27px;height: 23px;" src="{{arrowSrc}}"></image>
        </div>
    </div>
    <!-- weex not support  z-index -->
</template>

<style scoped>
    /* item height: 90 ; */
    .select-container{
        flex-direction: column;
        position: relative;
    }
    .scroller{
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 650px;
        opacity: 0;
    }
    .content{
        width: 750px;
        margin-top: 90px;
    }
    .mask{
        position: absolute;
        top: 90px;
        left: 0;
        bottom: 0;
        right: 0;
        background-color: rgba(0,0,0, .5);
        visibility: hidden;
        opacity:0.5;
    }
    .select {
        width: 750px;
        height: 90px;
        top:0;
        flex-direction: row;
        align-items: center;
        justify-content: space-between;
        padding-left: 30px;
        padding-right: 30px;
        border-bottom-width: 1;
        border-style: solid;
        border-color: #ddd;
        background-color: powderblue;
        position: absolute;
    }
    .options {
        position: relative;
        width: 750px;
        background-color: #ffffff;
        transform-origin: center center;
    }
    .cell {
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        width: 750px;
        height: 90px;
        padding-left: 30px;;
        padding-right: 30px;;

        border-bottom-width: 1;
        border-style: solid;
        border-color: #ddd;
    }
    .current-text {
        color: #333;
        font-size: 33px;
        flex: 1;
    }
    .name {
        color: #333;
        font-size: 33px;
        flex: 1;
    }
</style>


<script>
    var animation = weex.requireModule("animation");

    export default {
        data(){
            return{
                statusId: '0',
                statusType: 'jiaocai',
                top:'',
                status: [],
                jiaocai: [
                    {id: '0', name: '全部学院'},
                    {id: '1', name: '通信与信息工程学院'},
                    {id: '2', name: '通信抗干扰技术国家级重点实验室'},
                    {id: '3', name: '电子工程学院'},
                    {id: '4', name: '微电子与固体电子学院'},
                    {id: '5', name: '物理电子学院'},
                    {id: '6', name: '光电信息学院'},
                    {id: '7', name: '计算机科学与工程学院'},
                    {id: '8', name: '信息与软件工程学院'},
                    {id: '9', name: '自动化工程学院'},
                    {id: '10', name: '机械电子工程学院'},
                    {id: '11', name: '生命科学与技术学院'},
                    {id: '12', name: '数学科学学院'},
                    {id: '13', name: '经济与管理学院'},
                    {id: '14', name: '政治与公共管理学院'},
                    {id: '15', name: '外国语学院'},
                    {id: '16', name: '马克思主义教育学院'},
                    {id: '17', name: '能源科学与工程学院'},
                    {id: '18', name: '资源与环境学院'},
                    {id: '19', name: '航空航天学院'},
                    {id: '20', name: '格拉斯哥学院'},
                    {id: '21', name: '医学院'},
                    {id: '22', name: '英才试验学院'},
                    {id: '23', name: '创新创业学院'},
                    {id: '24', name: '基础与前沿学院'}
                ],
                jisuanji: [
                    {id: '0', name: '全部应用'},
                    {id: '1', name: '编程语言'},
                    {id: '2', name: '操作系统'},
                    {id: '3', name: '数据库'},
                    {id: '4', name: '办公软件'},
                    {id: '5', name: '图形图像'},
                    {id: '6', name: '网页制作'},
                    {id: '7', name: '考试认证'}
                ],
                kaoyan: [
                    {id: '0', name: '全部科目'},
                    {id: '1', name: '英语'},
                    {id: '2', name: '数学'},
                    {id: '3', name: '政治'},
                    {id: '4', name: '专业课'}
                ],
                waiyu: [
                    {id: '0', name: '全部考试'},
                    {id: '1', name: 'CET-4'},
                    {id: '2', name: 'CET-6'},
                    {id: '3', name: '雅思IELTS'},
                    {id: '4', name: '托福TOEFL'},
                    {id: '5', name: 'GRE'},
                    {id: '6', name: 'GMAT'},
                    {id: '7', name: '托业TOEIC'},
                    {id: '8', name: '英语专业四级'},
                    {id: '9', name: '英语专业八级'},
                    {id: '10', name: '其他语种考试'}
                ],
                gongwuyuan: [
                    {id: '0', name: '全部机构'},
                    {id: '1', name: '国家公务员'},
                    {id: '2', name: '地方公务员'}
                ],
                qita: [
                    {id: '0', name: '全部其他'},
                    {id: '1', name: '经管励志'},
                    {id: '2', name: '人文社科'},
                    {id: '3', name: '文学艺术'},
                    {id: '4', name: '科技生活'}
                ],
                flagSrc: 'https://gw.alicdn.com/tps/TB11a2lKFXXXXbVXpXXXXXXXXXX-32-32.png',
                arrowSrc: 'https://gw.alicdn.com/tps/TB1O3_aKFXXXXXdXVXXXXXXXXXX-27-23.png'
            }
        },
        created: function(){
            var self = this;
            if(self.statusType=='jiaocai') {
                self.status = self.jiaocai;
                self.top = -24 * 90 - 1;
            }
            if(self.statusType=='jisuanji') {
                self.status = self.jisuanji;
                self.top = -7 * 90 - 1;
            }
            if(self.statusType=='kaoyan') {
                self.status = self.kaoyan;
                self.top = -4 * 90 - 1;
            }
            if(self.statusType=='waiyu') {
                self.status = self.waiyu;
                self.top = -10 * 90 - 1;
            }
            if(self.statusType=='gongwuyuan') {
                self.status = self.gongwuyuan;
                self.top = -2 * 90 - 1;
            }
            if(self.statusType=='qita') {
                self.status = self.qita;
                self.top = -4 * 90 - 1;
            }
        },
        computed: {
            statusName: {
                get: function(){
                    var id = this.statusId;
                    return this.status.filter(function(s){
                        return s.id == id
                    })[0].name;
                }
            }
        },
        methods: {
            switchView: function() {
                this.toggleMaskVisible();
                this.opacity(this._ids.scroller.el.ref);
                this.collapse(this._ids.options.el.ref);
                this.rotate(this._ids.arrow.el.ref);
            },

            onItemClick: function(e) {
                var vid = e.target.attr.vid;
                this.updateStatus(vid);
                this.switchView();
                this.$dispatch('statuschange', {
                    id: this.statusId,
                    name: this.statusName
                })
            },

            updateStatus: function(id) {
                this.statusId = id;
            },

            toggleMaskVisible: function(){
                this.current_showMask = !this.current_showMask;
                var visibility = this.current_showMask? 'visible': 'hidden';
                this._ids.mask.el.setClassStyle({visibility:visibility});
            },

            collapse: function(ref, callback) {
                var translate = 'translate(0, 100%)';
                this.current_translate = this.current_translate ? '' : translate;
                this.anim(ref, {
                    transform: this.current_translate
                }, 'ease', 100, callback);
            },

            opacity: function(ref, callback) {
                var self = this;
                self.current_opacity = self.current_opacity === 1 ? 0 : 1;
                self.anim(ref, {
                    opacity: self.current_opacity
                }, 'ease', 100, callback);
            },

            rotate: function(ref, callback) {
                var self = this;
                if(!self.current_rotate) {
                    self.current_rotate = 0;
                }
                self.current_rotate = self.current_rotate + 180;
                self.anim(ref, {
                    transform: 'rotate(' + self.current_rotate + 'deg)'
                }, 'linear', 100, callback);
            },

            anim: function(ref, styles, timingFunction, duration, callback) {
                animation.transition(ref, {
                    styles: styles,
                    timingFunction: timingFunction,
                    duration: duration
                }, callback || function(){});
            }
        }
    }
</script>