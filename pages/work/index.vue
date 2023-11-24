<template>
    <view class="container">
<!--        <u-navbar :safeAreaInsetTop="false" :fixed="false" left-text="记一笔">
        </u-navbar> -->
        <view class="container-main">
            <!-- 顶部选项卡 -->
            <scroll-view class="pagecontrol-top-scroll" scroll-x="true" scroll-with-animation
                :scroll-into-view="scrollInto">
                <view class="pagecontrol-top-title">
                    <view v-for="(item, index) in itemsArr" :key="index" class="pagecontrol-top-text"
                        @click="changeTab(index)" :id="'tab' + index">
                        <view :class="tabIndex === index ? 'pagecontrol-top-selected' : 'pagecontrol-top-normal'">
                            {{ item }}
                        </view>
                        <view class="pagecontrol-bottom-line"
                            :class="tabIndex === index ? 'pagecontrol-bottom-line-show' : 'pagecontrol-bottom-line-visibility'">
                        </view>
                    </view>
                </view>
            </scroll-view>

            <!-- 内容 -->
            <view v-if="tabIndex != 2">
                <addInfo :types="types" :tabIndex="tabIndex"></addInfo>
            </view>
            <view v-if="tabIndex >= 2">
                <addRecord :types="types" :tabIndex="tabIndex"></addRecord>
            </view>
        </view>
    </view>
</template>
<script>
    import {
        getTypeList
    } from "@/api/system/redis";
    export default {
        data() {
            return {
                itemsArr: ['收入', '支出', '结账'],
                tabIndex: 0,
                scrollInto: '',
                scrollH: 660,
                types: [],
            };
        },
        computed: {
            computedTypes() {
                // 根据获取到的 types 数组来生成需要展示的内容
                // 示例：返回一个列表，用于渲染模板上的视图
                return this.types.map(type => ({
                    id: type.typeId,
                    typeName: type.typeName
                }));
            }
        },
        onPullDownRefresh() {
            setTimeout(function() {
                uni.stopPullDownRefresh(); //停止页面加载动画
            }, 1000);
        },
        methods: {
            initDate() {
                getTypeList().then(response => {
                    this.types = response.rows;
                });
            },
            // 监听滑动
            onChangeTab(e) {
                this.changeTab(e.detail.current);
            },
            // 切换选项
            changeTab(index) {
                if (this.tabIndex === index) {
                    return;
                }
                this.tabIndex = index;
                // 滚动到指定元素
                this.scrollInto = 'tab' + index;
            },
            rightClick() {
                console.log('rightClick');
            },
            leftClick() {
                uni.switchTab({
                    url: '../index'
                });
            }
        },
        onLoad() {
            uni.getSystemInfo({
                success: res => {
                    this.scrollH = res.windowHeight - 40;
                    // #ifdef MP
                    this.scrollH -= 44;
                    // #endif
                }
            });
            this.initDate();
        }
    };
</script>
<style>
    .container-main {
        /*       padding-left: 20px;
        padding-right: 20px; */
        background-color: #ffffff;
    }

    .pagecontrol-top-scroll {
        height: 40px;
        width: 100%;
        white-space: nowrap;
        box-sizing: border-box;
        border-bottom-width: 1rpx;
        border-bottom-style: solid;
        border-bottom-color: #ededed;
    }

    .pagecontrol-top-title {
        height: 100%;
        width: 100%;
        display: flex;
        justify-content: space-around;
    }

    .pagecontrol-top-text {
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        text-align: center;
    }

    .pagecontrol-top-normal {
        color: black;
        font-size: 16px;
    }

    .pagecontrol-top-selected {
        color: #0abafa;
        font-size: 17px;
    }

    .pagecontrol-bottom-line {
        width: 100%;
        height: 1px;
        margin-top: 2px;
        background-color: #0abafa;
    }

    .pagecontrol-bottom-line-show {
        visibility: visible;
    }

    .pagecontrol-bottom-line-visibility {
        visibility: hidden;
    }

    .order-item {
        background-color: #ffffff;
        padding: 20rpx;
        border-radius: 15rpx;
        margin: 15rpx;
    }

    .order-top-view {
        display: flex;
        justify-content: space-between;
        margin-bottom: 10rpx;
    }

    .order-store-name {
        font-size: 16px;
        font-weight: 500;
    }

    .order-status {
        font-size: 12px;
        color: #a5a5a5;
    }

    .order-content {
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .order-date {
        margin: 6rpx 0;
    }

    .bottom-buttons {
        margin-top: 10rpx;
        display: flex;
        justify-content: flex-end;
        align-items: center;
    }

    .button-tag {
        margin-left: 40rpx;
        height: 30px;
        line-height: 30px;
        padding: 0 20px;
        border-radius: 15px;
        color: #a5a5a5;
        border: 1px solid #ededed;
    }
</style>