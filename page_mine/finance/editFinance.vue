<template>
    <view class="container">
        <uni-nav-bar right-text="查询" :border="false" @clickRight="clickRight()"/>
        <!-- <u-navbar rightText="查询" @rightClick="clickRight()" :safeAreaInsetTop="false"></u-navbar> -->
        <view class="text-area">
            <view class="u-page">
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
        <u-popup customStyle="width:300px" :show="show" @close="close" @open="open" mode="right">
            <view>
                <u-form :model="form" ref="uForm">
                    <u-form-item labelWidth="30%" prop="type" borderBottom @click="clickType" label="所属表格">
                        <u--input v-model="form.type" disabled disabledColor="#ffffff" placeholder="请选择所属表格"
                            border="none"></u--input>
                        <u-icon slot="right" name="arrow-right"></u-icon>
                    </u-form-item>
                    <u-form-item labelWidth="30%" prop="recordInfo" @click="changeRecord" borderBottom label="结账信息">
                        <u--input v-model="form.dec" disabled disabledColor="#ffffff" placeholder="请选择信息"
                            border="none"></u--input>
                        <u-icon slot="right" name="arrow-right"></u-icon>
                    </u-form-item>
                </u-form>
                <view class="search">
                    <u-button @click="resetInfo" type="warning" :customStyle="buttonStyle">
                        重置
                    </u-button>
                    <u-button type="primary" @click="submitInfo" :customStyle="buttonStyle">
                        确定
                    </u-button>
                </view>
            </view>
            <my-picker mode="selector" @confirm="infoConfirm" v-model="showRecord" :default-selector="[0]"
                :range="recordListBak" range-key="value">
                <template v-slot>
                    <uni-search-bar placeholder="请输入信息" :focus="false" v-model="recordSerach" cancelButton="none"
                        @input="controlInput" style=" width: 70%;">
                    </uni-search-bar>
                </template>
            </my-picker>
            
            <u-picker :defaultIndex="[0]" :show="showType" :columns="computedTypes" @cancel="typeCancel" @close="typeCancel"
                @confirm="typeConfirm" closeOnClickOverlay @change="typeChange" keyName="text">
            </u-picker>
        </u-popup>

       
    </view>

</template>

<script>
    import {
        getStaticInfo,
        getTableInfo
    } from "@/api/system/static";
    import {
        getTypeList
    } from "@/api/system/redis"
    import {
        listFinanceAll
    } from "@/api/system/finance";
    export default {
        data() {
            return {
                showRecord: false,
                showType: false,
                show: false,
                typeId: null,
                financeId: null,
                recordSerach: "",
                types: [],
                form: {},
                query: {},
                pageNum: 1,
                pageSize: 20,
                total: 0,
                nowNum: 0,
                type: "",
                title: "",
                info: {},
                indexList: [],
                recordList: [],
                recordListBak: [],
                scrollTop: 0,
                showBackTop: false, // 控制u-back-top按钮的显示与隐藏
                buttonStyle:{
                    marginLeft: '30rpx',
                    marginRight: '30rpx',
                    width: '100px'
                }
            }
        },
        computed: {
            computedTypes() {
                return [this.types];
            }
        },
        onNavigationBarButtonTap(e) {
            
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
            clickRight(){
                this.show = !this.show
            },
            changeItem(e) {
                console.log(e)
                this.query.financeIds = e.id
            },
            changeRecord(e) {
                let query = {}
                query.financeType = this.form.typeId
                listFinanceAll(query).then(response => {

                    const savedData = response.rows.map(data => ({
                        id: data.financeId,
                        value: data.financeDec,
                        financeFlag: data.financeFlag
                    }));
                    this.recordList = savedData.filter(item => item.financeFlag === '1');
                    this.recordListBak = this.recordList;
                    this.showRecord = true;
                })
            },
            clickType() {
                this.showType = true;
            },
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
                    url: '/page_mine/finance/editFinanceInfo?financeIds=' + event.financeId
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
            },
            open() {
                // console.log('open');
            },
            close() {
                this.show = false
                this.typeId = null
                this.financeId = null
                this.resetInfo();
            },
            typeCancel(e) {
                console.log(e)
                this.showType = false
            },
            resetInfo() {
                this.reset()
                this.form = {}
                this.recordList = []
                this.recordListBak = []
                this.loadmore()
                this.show = false
            },
            submitInfo() {
                this.reset();
                let query = {}
                query.pageNum = this.pageNum;
                query.pageSize = this.pageSize;
                query.financeType = this.typeId;
                query.financeId = this.financeId
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
                    this.show = false
                })
            },
            typeChange(e) {
                console.log(e)
            },
            typeConfirm(e) {
                this.form.type = e.value[0].text
                this.form.typeId = e.value[0].value
                this.form.dec = ""
                this.typeId = e.value[0].value
                this.query.financeType = e.value[0].value
                this.showType = false
            },
            //查找对应的信息
            infoConfirm(e) {
                this.financeId = this.recordListBak[e[0]].id;
                this.form.dec = this.recordListBak[e[0]].value;
                console.log(this.financeId)
            },
            //处理输入框事件
            controlInput() {
                this.recordListBak = this.recordSerach ? this.recordList.filter(item => item.value.includes(this
                    .recordSerach)) : this.recordList;
            },
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

    .serach-info {
        display: flex;
    }

    .search {
        padding-top: 10rpx;
        display: flex;
        justify-content: space-around;
    }

    .custom-style {
        margin-left: 20rpx;
        margin-right: 20rpx;
        width: 100px
    }
</style>