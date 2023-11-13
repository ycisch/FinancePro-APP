<template>
    <view class="container">
        <view class="text-area">
            <view>
                <PanelGroup :info="info" @handleSetLineChartData="handleSetLineChartData"></PanelGroup>
                <!-- <barChart :type="type"></barChart> -->
                <!-- <barChart></barChart> -->
            </view>
            <view class="u-page">
                <u-list @scrolltolower="scrolltolower">
                    <u-list-item v-for="(item, index) in indexList" :key="index">
                        <view class="info">
                            <view class="top">
                                <view class="date">{{formatDate(item.financeCreate)}}</view>
                                <!-- <view class="type"><u-tag size="mini" :text="item.sysType.typeName"></u-tag></view> -->
                            </view>
                            <view class="item-box">
                                <view class="content">{{item.financeDec}}</view>
                                <view class="amount">
                                    <view>收入<span
                                            style="color: crimson;">{{item.financeIncome == null?"0.00":formatNumber(item.financeIncome)}}</span>
                                    </view>
                                    <view>支出<span
                                            style="color: seagreen;">{{item.financeExpenditure == null?"0.00":formatNumber(item.financeExpenditure)}}</span>
                                    </view>
                                </view>
                            </view>
                        </view>
                    </u-list-item>
                </u-list>
            </view>
        </view>
    </view>
</template>


<script>
    import {
        getStaticInfo,
        getTableInfo,
        getStaticInfoList
    } from "@/api/system/static";
    import row from "../uni_modules/uview-ui/libs/config/props/row";

    const info = {
        "expend": {
            name: "支出"
        },
        "income": {
            name: "收入"
        },
        "shangdong": {
            name: "山东"
        },
        "xinjiang": {
            name: "新疆"
        }
    }

    export default {
        data() {
            return {
                pageNum: 1,
                pageSize: 20,
                total: 0,
                nowNum: 0,
                type: "",
                title: "",
                info: {},
                indexList: [],
            }
        },
        mounted() {
            this.getStaticInfo();
        },
        onLoad() {
            this.loadmore()
        },
        onPullDownRefresh() {
            console.log('refresh');
            this.reset();
            this.getStaticInfo();
            this.loadmore();

            setTimeout(function() {
                uni.stopPullDownRefresh(); //停止页面加载动画
            }, 1000);
        },
        methods: {
            reset() {
                this.pageNum = 1;
                this.pageSize = 20;
                this.total = 0;
                this.nowNum = 0;
                this.indexList = [];
                this.info = {};
            },
            handleSetLineChartData(type) {
                this.type = type;
                this.title = info[type].name;
                if (type == 'expend' || type == 'income') {
                    this.handleToChartInfo(type)
                } else if(type == 'shangdong') {
                    this.handleToLineChartInfo(type)
                }
            },
            getStaticInfo() {
                getStaticInfo().then(respone => {
                    this.info = respone;
                })
            },
            scrolltolower() {
                this.pageNum = this.pageNum + 1;
                this.loadmore()
            },
            loadmore() {
                let query = {}
                query.pageNum = this.pageNum;
                query.pageSize = this.pageSize;
                getTableInfo(query).then(res => {
                    this.total = res.total;
                    this.nowNum = this.nowNum + res.rows.length;
                    if (this.nowNum > this.total) {
                        uni.showToast({
                            title: '无最新数据',
                            icon: 'none'
                        });
                        return;
                    }
                    this.indexList = this.indexList.concat(res.rows);
                })
            },
            handleToChartInfo(type) {
                uni.navigateTo({
                    url: '/page_view/static?type=' + type
                })
            },
            handleToLineChartInfo(type){
                uni.navigateTo({
                    url: '/page_view/line?type=' + type
                })
            },
            formatDate(value) {
                // 假设value的值为 "2023-11-09"
                let date = new Date(value);

                let formattedDate =
                    `${date.getFullYear()}年${date.getMonth() + 1}月${date.getDate()}日 ${this.getWeekDay(date)}`;
                return formattedDate;
            },
            getWeekDay(date) {
                let weekdays = ["星期日", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"];
                return weekdays[date.getDay()];
            },
            formatNumber(num) {
                return num.toFixed(2);
            }
        }

    }
</script>

<style scoped>
    .container {
        background-color: #f9f9f9;
    }

    .u-page {
        margin-top: 20rpx;
        margin-left: 10rpx;
        margin-right: 10rpx;
        border-radius: 3%;
        background-color: #ffffff;
    }

    .u-list-item:not(:last-child) {
        border-bottom: 1px solid #e5e5e5;
    }
    
    .top{
        display: flex;
    }

    .date {
        font-size: 12px;
        padding-left: 10rpx;
        padding-top: 5rpx;
        color: #888;
        flex: 0.9;
        /* 灰色 */
    }
    
    .type{
        font-size: 5px;
    }

    .content {
        width: 70%;
        padding-left: 10rpx;
        font-size: 20px;
        display: flex;
        align-items: center; /* 垂直居中 */
        /* 根据你的需求设置样式，可能需要设置宽度等属性 */
    }

    .amount {
        /* 根据你的需求设置样式，可能需要设置宽度等属性 */
        margin-left: 20rpx;
        font-size: 20px;
        display: flex;
        flex-direction: column;
        text-align: left;
    }

    .item-box {
        display: flex;
    }
</style>