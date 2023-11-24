<template>
    <view class="group-info">
        <view class="group-list">
            <view class="item" @click="handleSetLineChartData('xinjiang')">
                <view class="icon icon-hetao"></view>
                <view class="item-info">
                    <view class="item-font">新疆核桃</view>
                    <view class="item-money">
                        <span style="font-weight: bold;">暂无统计</span>
                        <!-- <u-count-to :start-val="0" :end-val="Number(info.expendAllInfo)" :duration="2600" bold /> -->
                    </view>
                </view>
            </view>
            <view class="item" @click="handleSetLineChartData('shangdong')">
                <view class="icon icon-shouru"></view>
                <view class="item-info">
                    <view class="item-font">山东收入</view>
                    <view class="item-money"><u-count-to :start-val="0" :end-val="Number(info.incomeAllInfo)"
                            :duration="3000" bold /></view>
                </view>
            </view>
        </view>
        <view class="group-list">
            <view class="item" @click="handleSetLineChartData('income')">
                <view class="icon icon-shouru1"></view>
                <view class="item-info">
                    <view class="item-font">收入</view>
                    <view class="item-money"><u-count-to :start-val="0" :end-val="Number(info.incomeAll)"
                            :duration="3200" bold /></view>
                </view>
            </view>
            <view class="item" @click="handleSetLineChartData('expend')">
                <view class="icon icon-zhichu"></view>
                <view class="item-info">
                    <view class="item-font">支出</view>
                    <view class="item-money"><u-count-to :start-val="0" :end-val="Number(info.expendAll)"
                            :duration="3600" bold /></view>
                </view>
            </view>
        </view>
    </view>
</template>
<script>
    export default {
        props: {
            info: {
                type: Object,
            },
        },
        data() {
            return {}
        },
        methods: {
            handleSetLineChartData(type) {
                this.$emit('handleSetLineChartData', type)
            },
        }
    }
</script>
<style scoped>
    .group-list {
        display: flex;
    }

    .item {
        display: flex !important;
        background-color: #D8E5FF !important;
        margin-left: 10rpx;
        margin-right: 10rpx;
        margin-top: 20rpx;
        padding-top: 10rpx;
        padding-left: 15rpx;
        padding-bottom: 10rpx;
        background-color: aqua;
        width: 50%;
        box-sizing: border-box;
        border-radius: 7px;
        text-align: center;
    }

    .item-info {
        /* display: flex; */
        font-size: 15px;
    }

    .item-money {
        font-weight: bold;
    }
</style>