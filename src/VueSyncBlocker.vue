<style>
    .vue-sync-blocker-mask {
        width: 100%;
        height: 100%;
        background: rgba(0, 0, 0, 0.1);
        position: absolute;
        top: 0px;
        left: 0px;
    }

    .vue-sync-blocker-msg {
        overflow: hidden;
        position: relative;
    }
</style>
<template>
    <div class="vue-sync-blocker-mask" :transition="transition" v-show="show">

    </div>
    <div class="vue-sync-blocker-msg" v-show="show" :transition="msg_transition">
        <slot></slot>
    </div>
</template>


/*!
* vue-loader v0.0.1 (https://github.com/johnnyGoo/vue-loader)
* Author: Johnny chen
*
* Copyright 2013-2015 Johnny chen
*/
<script>


    var count = 0;

    if (!window.Smart) {
        throw 'SyncBlocker required smart.js'
    }
    var Smart = window.Smart;
    var Css = Smart.Css;
    var Loader = Smart.Loader;
    var _ = Smart._;
    // 注册
    export default {
        // 声明 props
        props: {
            show: {
                type: Boolean,
                default: false
            },
            normal: {
                type: Object,
                default: function () {
                    return {opacity: 1, transition: 'all 0.3s ease'}
                }
            },
            enter: {
                type: Object,
                default: function () {
                    return {opacity: 0}
                }
            },
            leave: {
                type: Object,
                default: function () {
                    return {opacity: 0}
                }

            },
            msgNormal: {
                type: Object,
                default: function () {
                    return {opacity: 1, transition: 'all 0.3s ease'}
                }
            },
            msgEnter: {
                type: Object,
                default: function () {
                    return {opacity: 0}
                }
            },
            msgLeave: {
                type: Object,
                default: function () {
                    return {opacity: 0}
                }

            }
        },
        data: function () {
            count++;
            return {
                transition: 'vue-sync-blocker-count' + count, msg_transition: 'vue-sync-blocker-msg-count' + count,
                startTime: 0

            }
        },

        methods: {
            maxPercent: function () {
                return (new Date().getTime() - this.startTime) / this.minDuration
            },
            loadingComplete: function () {
                var me = this;
                me.$emit('loading-complete', me.loader);
                clearInterval(me.iv);
            },
            startLoading: function () {
                var me = this;
                me.startTime = new Date().getTime();

                me.loader = new Loader();
                me.loader.addImages(me.assets);
                me.$emit('loading-start');
                //监听资源加载进度事件
                me.loader.addProgressListener(function (p) {
                    me.$emit('loading-progress', {
                        completedCount: p.completedCount,
                        percent: Math.min(p.completedCount / p.totalCount, me.maxPercent()),
                        totalCount: p.totalCount
                    });
                });
                //监听资源加载完成事件
                me.loader.addCompletionListener(function () {
                    me.iv = setInterval(function () {
                        me.$emit('loading-progress', {percent: me.maxPercent()});
                        if (me.maxPercent() >= 1) {
                            me.loadingComplete()
                        }
                    }, 30);
                    //显示图片
                });
                //启动资源加载管理器
                me.loader.start();
            }
        },
        computed: {},


        ready: function () {
            if (_.isEqual({}, this.enter) && _.isEqual({}, this.leave)) {
                Css.smartCss(this.$el, this.normal, 'px');
                return;
            }
            if (_.isEqual({}, this.enter)) {
                this.enter = _.clone(this.leave);
            } else if (_.isEqual({}, this.leave)) {
                this.leave = _.clone(this.enter);
            }
            Css.createSmartCssStyle('.' + this.transition + '-transition', _.clone(this.normal), 'px');
            Css.createSmartCssStyle('.' + this.transition + '-leave', _.clone(this.leave), 'px');
            Css.createSmartCssStyle('.' + this.transition + '-enter', _.clone(this.enter), 'px');

            Css.createSmartCssStyle('.' + this.msg_transition + '-transition', _.clone(this.msgNormal), 'px');
            Css.createSmartCssStyle('.' + this.msg_transition + '-leave', _.clone(this.msgLeave), 'px');
            Css.createSmartCssStyle('.' + this.msg_transition + '-enter', _.clone(this.msgEnter), 'px');


        }


    }


</script>