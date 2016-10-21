<template>
    <div class="dialog-mask" v-if="mask && display" @click="close"></div>
    <div :id="id" class="dialog-wrapper" v-show="display" transition="expand">
        <div :class="['dialog-head', type>2&&title?'prompt-head':'alert-head']">
            <div class="dialog-title" v-if="type>2&&title">
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
                id: new Date().getTime().toString() + Math.random().toString().slice(2),
                type: 1,
                display: false,
                title: '提示',
                msg: '未选中数据',
                submitCb: null,
                submitParams: null,
                btnName: ['确定', '取消', '关闭'],
                orientation: true
            }
        },
        props: {
            config: {
                type: Object,
                default() {
                    return {
                        type: 1, // 1.关闭，2.确定取消3.标题确定取消，>3自定义
                        display: false,
                        title: '提示',
                        msg: '未选中数据',
                        submitCb: null,
                        submitParams: null,
                        btnName: ['确定', '取消', '关闭'],
                        width: ['318px', '384px', '512px'],
                        height: ['138px', '198px', '280px'],
                    }
                }
            },
            dialog: {
                type: Object,
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
                    this.animate(this.wrapper);
                    this.display = true;
                    this.$nextTick(function () {
                        this._reLayout(this.wrapper);
                        this._resetPosition(this.wrapper);
                    });
                }
            },
            fullfilled() {
                if (this.fullfilled) {
                    this.close();
                }
            }
        },
        methods: {
            mixinConfig() {
                if (!this.dialog) {
                    return;
                }
                let config = Object.assign({}, this.config, this.dialog);
                let self = this;
                Object.keys(config).forEach(key => {
                    switch (key) {
                        case 'btnName':
                            if (self.dialog.btnName && self.dialog.btnName.length > 0) {
                                self.btnName = config.btnName;
                            } else {
                                if (config.type === 1) {
                                    self.btnName = [self.config.btnName[2]];
                                }
                                if (config.type > 1) {
                                    self.btnName = [self.config.btnName[1], self.config.btnName[0]];
                                }
                            }
                        case 'display':
                        case 'width':
                        case 'height':
                            return;
                        default:
                            this[key] = config[key];
                    }
                });
//                console.log('mixinConfig(data: %s)', JSON.stringify(this.$data));
            },
            _reLayout(wrapper) {
                if (!wrapper || !this.dialog) {
                    return;
                }

                if (this.dialog.width) {
                    wrapper.style.width = this.dialog.width;

                    let w = parseInt(this.dialog.width.slice(0, -2));
                    let maxW = Math.ceil(document.documentElement.clientWidth / 2);

                    if (w > maxW || wrapper.offsetWidth > maxW) {
                        wrapper.style.width = maxW + 'px';
                    }
                } else {
                    switch (this.type) {
                        case 1:
                        case 2:
                            wrapper.style.width = this.config.width[0];
                            break;
                        case 3:
                            wrapper.style.width = this.config.width[1];
                            break;
                        default:
                            wrapper.style.width = this.config.width[2];
                    }
                }

                if (this.dialog.height) {
                    wrapper.style.height = this.dialog.height;

                    let h = parseInt(this.dialog.height.slice(0, -2));
                    let maxH = Math.ceil(document.documentElement.clientHeight / 2);
                    if (h > maxH || wrapper.offsetHeight > maxH) {
                        this.body.style.height = maxH - this.head.offsetHeight - this.foot.offsetHeight - 2 + 'px';
                        this.body.style['overflow-y'] = 'auto';
                    }
                } else {
                    switch (this.type) {
                        case 1:
                        case 2:
                            wrapper.style.height = this.config.height[0];
                            break;
                        case 3:
                            wrapper.style.height = this.config.height[1];
                            break;
                        default:
                            wrapper.style.height = this.config.height[2];
                    }
                }
            },
            _resetPosition(wrapper) {
                if (wrapper) {
                    wrapper.style.left = (document.documentElement.clientWidth - wrapper.offsetWidth) / 2 + 'px';
                    wrapper.style.top = (document.documentElement.clientHeight - wrapper.offsetHeight) / 2 + 'px';
                }
            },
            animate(wrapper) {
                if (this.orientation) {
                    wrapper.style.left = document.documentElement.clientWidth + 'px';
                    wrapper.style.top = 0;
                } else {
                    wrapper.style.left = 0;
                    wrapper.style.top = 0;
                }
                this.orientation = !this.orientation;
            },
            close() {
                this.body.style['overflow-y'] = 'visible';
                this.body.style.width = 'auto';
                this.body.style.height = 'auto';
                this.display = false;
            },
            submit() {
                this.submitCb && this.submitCb(this.submitParams);
                if (this.fullfilled) {
                    this.close();
                }
            }
        },
        created() {
            this.mixinConfig();
        },
        ready() {
            this.wrapper = document.getElementById(this.id);
            this.head = this.wrapper.querySelector('.dialog-head');
            this.foot = this.wrapper.querySelector('.dialog-foot');
            this.body = this.wrapper.querySelector('.dialog-body');
            this.display = true;
            this.$nextTick(function () {
                this.animate(this.wrapper);
                this._reLayout(this.wrapper);
                this._resetPosition(this.wrapper);
            });

            window.onresize = function () {
                this._resetPosition(this.wrapper);
            }.bind(this);
        }
    };
</script>
<style scoped>
    .dialog-mask {
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
    }

    .alert-head {
        background-color: #fff;
        height: 28px;
    }

    .prompt-head {
        background-color: #ececec;
        height: 40px;
    }

    .dialog-title {
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

    .dialog-foot {
        height: 70px;
        padding: 18px 0 0;
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

    .expand-transition {
        transition: all .3s ease;
        opacity: 0.96;
    }

    .expand-enter, .expand-leave {
        opacity: 0;
    }
</style>
