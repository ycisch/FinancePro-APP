<template>
    <view class="container">
        <view class="text-area">
            <view>
                <PanelGroup :info="info" @handleSetLineChartData="handleSetLineChartData"></PanelGroup>
                <!-- <barChart :type="type"></barChart> -->
                <!-- <barChart></barChart> -->
            </view>

            <view class="u-page">
                <view class="serach-info">
                    <view style="width: 100px;"><uni-section title="表格信息" type="line"></uni-section></view>
                     <view style="width: 300px;"><uni-data-select v-model="typeId" :localdata="types" @change="change"></uni-data-select></view>
                </view>
                <u-list @scrolltolower="scrolltolower">
                    <u-list-item v-for="(item, index) in indexList" :key="index" class="custom-list-item">
                        <view class="info" @click="handleListClick(item)">
                            <view class="top">
                                <view class="date">{{formatDate(item.financeCreate)}}</view>
                                <view class="type">
                                    <view v-if="item.financeFlag == '0'">
                                        <u-text size="mini" text="结清" type="success"></u-text>
                                    </view>
                                    <view v-if="item.financeFlag == '1'">
                                        <!-- <u-text size="mini" text="未结" type="primary"></u-text>  -->
                                    </view>
                                </view>
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
        getTableInfo
    } from "@/api/system/static";
    import {
        getTypeList
    } from "@/api/system/redis";

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
                typeId: null,
                types: [],
                pageNum: 1,
                pageSize: 20,
                total: 0,
                nowNum: 0,
                type: "",
                title: "",
                info: {},
                indexList: [],
                scrollTop: 0,
                showBackTop: false, // 控制u-back-top按钮的显示与隐藏
            }
        },
        mounted() {
            this.getStaticInfo();
        },
        onLoad() {
            this.initDate();
            this.loadmore()
        },
        onPullDownRefresh() {
            console.log('refresh');
            this.reset();
            this.info = {};
            this.getStaticInfo();
            this.loadmore();

            setTimeout(function() {
                uni.stopPullDownRefresh(); //停止页面加载动画
            }, 1000);
        },
        methods: {
            initDate() {
                getTypeList().then(response => {
                    // 使用 map 函数进行转换
                    this.types = response.rows.map((row, index) => {
                        return {
                            value: row.typeId,
                            text: row.typeName
                        };
                    });
                });
            },
            change(e) {
                this.typeId = e
                this.reset();
                this.loadmore();
            },
            handleListClick(event) {
                console.log(event)
                uni.navigateTo({
                    url: '/page_work/financeInfo?financeIds=' + event.financeId
                })
            },

            reset() {
                this.pageNum = 1;
                this.pageSize = 20;
                this.total = 0;
                this.nowNum = 0;
                this.indexList = [];
            },
            handleSetLineChartData(type) {
                this.type = type;
                this.title = info[type].name;
                if (type == 'expend' || type == 'income') {
                    this.handleToChartInfo(type)
                } else if (type == 'shangdong') {
                    this.handleToLineChartInfo(type)
                }
            },
            getStaticInfo() {
                getStaticInfo().then(respone => {
                    this.info = respone;
                })
            },

            // handleScroll(e) {
            //     this.scrollTop = e; // 监听uList组件的scroll事件，更新scrollTop的值
            //     this.showBackTop = e > 1000; // 当滚动超过1000px时显示u-back-top按钮
            //     console.log(this.scrollTop + "  " + this.showBackTop)
            // },
            // handleBackTopClick() {
            //     console.log("####")
            //     // 处理点击u-back-top按钮的逻辑，将页面滚动到顶部
            //     uni.pageScrollTo({
            //         scrollTop: '.u-page',
            //         duration: 300,
            //     });
            // },
            scrolltolower() {
                this.pageNum = this.pageNum + 1;
                this.loadmore()
            },
            loadmore() {
                let query = {}
                query.pageNum = this.pageNum;
                query.pageSize = this.pageSize;
                query.financeType = this.typeId;
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
            handleToLineChartInfo(type) {
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

    .info {
        border-bottom: 1px solid #e5e5e5;
    }

    .top {
        display: flex;
    }

    .date {
        font-size: 12px;
        padding-left: 10rpx;
        padding-top: 5rpx;
        color: #888;
        /* 灰色 */
    }

    .type {
        font-size: 5px;
        margin-left: 20rpx;
    }

    .content {
        width: 70%;
        padding-left: 10rpx;
        font-size: 15px;
        display: flex;
        align-items: center;
        /* 垂直居中 */
        /* 根据你的需求设置样式，可能需要设置宽度等属性 */
    }

    .amount {
        /* 根据你的需求设置样式，可能需要设置宽度等属性 */
        margin-left: 20rpx;
        font-size: 15px;
        display: flex;
        flex-direction: column;
        text-align: left;
    }

    .item-box {
        display: flex;
    }
    .serach-info{
        display: flex;
    }
</style>