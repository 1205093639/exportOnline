<template>
    <div class='exportOnline'>
        <div class="btn">
            <el-button type="primary" :disabled="show" icon="el-icon-download" @click.native="down(options.types[0])" v-if="options.types.length==1"> {{options.types[0].label||'导出'}}</el-button>
            <el-dropdown :disabled="show" v-else>
                <el-button type="primary">
                    导出<i class="el-icon-arrow-down el-icon--right"></i>
                </el-button>
                <el-dropdown-menu slot="dropdown">
                    <el-dropdown-item icon="el-icon-download" v-for='(e,i) in options.types' :key='i' @click.native="down(e)">{{e.label}}</el-dropdown-item>
                </el-dropdown-menu>
            </el-dropdown>
        </div>
        <div class="downBox" id="downBox" :class="{halfHide}" v-if="show">
            <div class="exportOnlineIcon">
                <i :class="[halfHide?'el-icon-full-screen':'el-icon-minus']" @click="change"></i>
            </div>
            <el-progress class="per" :type="halfHide?'circle':'line'" :text-inside="!halfHide" :stroke-width="20" :percentage="percentage" :color="customColors"></el-progress>
            <div class="tip">
                正在导出下载中....<br>请不要切换/关闭本页面
            </div>
        </div>
    </div>
</template>
<script>
export default {
    name: 'exportOnline',
    props: {
        options: {//下载选项
            type: Object,
            default: () => {
                return {
                    apiName: 'GetALlWork_Center',//导出接口名
                    types: [
                        { value: 0, label: '导出当前页' },
                        { value: 1, label: '导出搜索数据' },
                        { value: 2, label: '导出全部数据' },
                    ]

                }
            },
        },
        otherData: {//搜索条件及页码尺寸
            type: Object,
            default: () => {
                return {//搜索条件及页码尺寸
                }
            },
        },
    },

    data() {
        return {
            halfHide: false,
            show: false,
            percentage: 0,
            guid: '',
            customColors: [
                { color: '#f56c6c', percentage: 20 },
                { color: '#e6a23c', percentage: 40 },
                { color: '#5cb87a', percentage: 60 },
                { color: '#1989fa', percentage: 80 },
                { color: '#6f7ad3', percentage: 100 }
            ]
        }
    },
    methods: {
        DownloadFn(url) {
            setTimeout(() => {
                this.guid = ''
                var a = document.createElement('a')
                a.href = url
                a.download = url
                a.click()
                this.show = false
                this.$message.success('导出成功');
                this.percentage = 0
            }, 1000)
        },
        change() {
            this.halfHide = !this.halfHide
            var dv = document.getElementById('downBox');
            if (this.halfHide) {
                dv.style.left = '1%';
                dv.style.top = '1%';
            } else {
                dv.style.left = '40%';
                dv.style.top = '20%';
            }

        },
        down(e) {
            this.show = true
            this.starDown(e)
            this.$nextTick(() => {
                this.move()
            })
        },
        starDown(e) {
            let data = {
                otherData: this.otherData,
                type: e.value,
                guid: this.guid
            }
            this.$apis[this.options.apiName](data).then((res) => {
                this.percentage = res.percentage || 0
                this.guid = this.guid || res.guid || ''
                console.log(res, data, '请求下载数据');
                res.url ? this.DownloadFn(res.url) : this.timer = setTimeout(() => this.starDown(e), 1000);
            }).catch((err) => {
                this.show = false
                this.guid = ''
                this.$message.error(err || '下载失败');
            })
        },
        move() {
            //获取元素
            var dv = document.getElementById('downBox');
            var x = 0;
            var y = 0;
            var l = 0;
            var t = 0;
            var isDown = false;
            //鼠标按下事件
            dv.onmousedown = function (e) {
                //获取x坐标和y坐标
                x = e.clientX;
                y = e.clientY;

                //获取左部和顶部的偏移量
                l = dv.offsetLeft;
                t = dv.offsetTop;
                //开关打开
                isDown = true;
                //设置样式  
                dv.style.cursor = 'move';
                dv.style.transition = '';

            }
            //鼠标移动
            window.onmousemove = function (e) {
                if (isDown == false) {
                    return;
                }
                //获取x和y
                var nx = e.clientX;
                var ny = e.clientY;
                //计算移动后的左偏移量和顶部的偏移量
                var nl = nx - (x - l);
                var nt = ny - (y - t);
                dv.style.left = nl + 'px';
                dv.style.top = nt + 'px';
            }
            //鼠标抬起事件
            dv.onmouseup = function () {
                //开关关闭
                isDown = false;
                dv.style.cursor = 'default';
                dv.style.transition = 'all .5s';

            }
        }
    },
    mounted() {

    },
    beforeDestroy() {
        clearTimeout(this.timer)

    },
}
</script>
<style lang='scss' scoped>
.exportOnline {
    display: inline-block;
    .downBox {
        .exportOnlineIcon {
            position: absolute;
            right: 0;
            top: 0;
            padding: 5px 10px;
            &:hover {
                background: #e5e9f2;
                transition: all 0.5s;
            }
        }
        z-index: 999;
        position: fixed;
        right: 40%;
        top: 20%;
        border: 1px solid #ccc;
        box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
        background: #fff;
        border-radius: 4px;
        padding: 30px;
        text-align: center;
        width: 500px;
        font-size: 12px;
        color: #ff6600;
        -webkit-touch-callout: none;

        -webkit-user-select: none;

        -khtml-user-select: none;

        -moz-user-select: none;

        -ms-user-select: none;

        user-select: none;
        .tip {
        }
    }
    .halfHide {
        right: 1%;
        top: 1%;
        width: 130px;
        padding: 5px;

        ::v-deep .per {
            .el-progress-circle {
                width: 80px !important;
                height: 80px !important;
            }
            .el-progress__text {
                font-size: 12px !important;
            }
        }
    }
    .btn {
    }
}
</style>