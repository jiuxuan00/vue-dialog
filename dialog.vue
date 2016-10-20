<template>
    <div class="mask" v-if="mask && display" @click="close"></div>
    <div class="dialog-wrapper" v-show="display" transistion="dialog" transition-mode="in-out">
        <div :class="[title?'dialog-head':'alert-head']">
            <div class="dialog-title" v-if="title">
                {{title}}
            </div>
            <div class="dialog-close">
                <img src="../../static/img/fork.png" @click="close">
            </div>
        </div>
        <div class="dialog-body">
            <slot>
                <p :class="{'alert-msg':type==1, 'dialog-msg':type==2, 'dialog-data':type==3}">{{msg}}</p>
            </slot>
        </div>
        <div class="dialog-foot">
            <a href="javascript:;" class="dialog-submit" @click="submit" v-if="btnName[1]">{{btnName[1]}}</a>
            <a href="javascript:;" class="dialog-cancel" @click="close">{{btnName[0]}}</a>
        </div>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                type: 1, // 1.关闭，2.确定取消3.标题确定取消，>3自定义
                display: false,
                title: '',
                msg: '未选中数据',
                submitCb: null,
                submitParams: null,
                btnName: ['确定', '取消', '关闭']
            }
        },
        props: {
            config: {
                type: Object,
                default() {
                    return {
                        type: 1, // 1、2.无标题无确定，3.有标题确定，>3有标题确定内容自定义
                        display: false,
                        title: '',
                        msg: '未选中数据',
                        submitCb: null,
                        submitParams: null,
                        btnName: ['确定', '取消', '关闭'],
                        width: [384, 512],
                        height: [198, 280],
                    }
                }
            },
            dialog: {
                type: Object,
                twoWay: true,
                required: true
            },
            mask: {
                type: Boolean,
                default: true
            },
            fullfilled: {
                type: Boolean,
                default: true
            }
        },
        watch: {
            dialog: {
                handler() {
                    this.mixinConfig();
                    this.$nextTick(() => {
                        this._resetPosition();
                    });
                    this.display = true;
                }
            }
        },
        methods: {
            mixinConfig() {
                console.log(`config=> ${JSON.stringify(this.config)}`);
                if (this.dialog) {
                    let config = Object.assign({}, this.config, this.dialog);
                    let self = this;
                    Object.keys(config).forEach(key => {
                        switch (key) {
                            case 'btnName':
                                console.log(`btnName=> ${JSON.stringify(config.btnName)}`);
                                if (self.dialog.btnName) {
                                    let len = self.dialog.btnName.length;
                                    if (len === 1 && config.type === 1) {
                                        self.btnName = config.btnName;
                                    }
                                    if (len > 1 && config.type > 1) {
                                        self.btnName = config.btnName;
                                    }
                                } else {
                                    if (config.type === 1) {
                                        self.btnName = [self.config.btnName[2]];
                                        console.log(`btnName=> ${JSON.stringify(self.btnName)}`);
                                    }
                                    if (config.type > 1) {
                                        self.btnName = [self.config.btnName[1], self.config.btnName[0]];
                                    }
                                }
                                console.log(`btnName=> ${JSON.stringify(self.btnName)}`);
                            case 'display': case 'width':  case 'height':
                                return;
                            default:
                                this[key] = config[key];
                        }
                    });
                }
            },
            close() {
                this.display = false;
            },
            submit() {
                this.submitCb && this.submitCb(this.submitParams);
                if (this.fullfilled) {
                    this.close();
                }
            },
            _resetPosition() {
                let wrapper = document.querySelector('.dialog-wrapper');
                if (!wrapper || !this.dialog) {
                    return;
                }

                if (this.dialog.width) {
                    wrapper.style.width = this.dialog.width
                } else {
                    if (this.type == 3) {
                        wrapper.style.width = this.config.width[0] + 'px';
                    } else if (this.type > 3) {
                        wrapper.style.width = this.config.width[1] + 'px';
                    }
                }

                if (this.dialog.height) {
                    wrapper.style.height = this.dialog.height
                } else {
                    if (this.type == 3) {
                        wrapper.style.height = this.config.height[0] + 'px';
                    } else if (this.type > 3) {
                        wrapper.style.height = this.config.height[1] + 'px';
                    }
                }

                wrapper.style.top = (document.documentElement.clientHeight - wrapper.offsetHeight) / 2 + 'px';
                wrapper.style.left = (document.documentElement.clientWidth - wrapper.offsetWidth) / 2 + 'px';
                console.log('height: %s\n width: %s\n', wrapper.offsetHeight, wrapper.offsetWidth)
            }
        },
        ready() {
            this.mixinConfig();
            this.$nextTick(function () {
                this._resetPosition();
            });
            this.display = true;
            let self = this;
            window.onresize = () => {
                self._resetPosition();
            }
        }
    };
</script>
<style scoped>
    .mask {
        position: fixed;
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        background-color: #000000;
        opacity: 0.6;
        z-index: 999;
    }

    .dialog-wrapper {
        position: fixed;
        z-index: 1000;
        border: solid 1px #2271ac;
        border-radius: 3px;
        text-align: center;
        background-color: #fff;
        opacity: .95;
        transition: all .3s ease;
        width: 318px;
        height: 138px;
    }

    .dialog-enter, .dialog-leave {
        opacity: 0;
    }

    .dialog-head {
        background-color: #ececec;
        height: 40px;
    }

    .alert-head {
        background-color: #fff;
        height: 28px;
    }

    .dialog-head .dialog-title {
        float: left;
        color: #000;
        font: 16px/2.6 "微软雅黑", arial, sans-serif;
        text-indent: 16px;
        font-weight: bold;
    }

    .dialog-close {
        cursor: pointer;
        float: right;
        line-height: 2.2;
        height: 100%;
        margin-right: 10px;
    }

    .dialog-body {
    }

    .dialog-foot {
        height: 80px;
        padding: 18px 0 30px;
        clear: both;
    }

    .dialog-submit, .dialog-cancel {
        display: inline-block;
        border: solid 1px #cccccc;
        border-radius: 2px;
        width: 96px;
        height: 32px;
        margin: 0;
        color: #666;
        font: 14px/2.1 "微软雅黑", arial, sans-serif;
    }

    .dialog-submit {
        background-color: #0196f0;
        margin-right: 10px;
        color: #fff;
    }

    .alert-msg, .dialog-msg, .dialog-data {
        text-align: center;
    }

    .alert-msg {
        color: #f00;
        font: bold 16px/2 "微软雅黑", arial, sans-serif;
    }

    .dialog-msg {
        color: #333;
        font: bold 16px/2 "微软雅黑", arial, sans-serif;
    }

    .dialog-data {
        margin: 40px 46px 12px;
    }
</style>
