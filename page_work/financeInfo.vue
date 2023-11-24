<template>
    <view>
        <view>
            <uni-card title="基本信息">
                <uni-table :border="true" stripe emptyText="暂无更多数据">
                    <!-- 表格数据行 -->
                    
                    <uni-tr>
                        <uni-td width="100">金额</uni-td>
                        <uni-td>{{financeInfo.financeIncome == null?financeInfo.financeExpenditure:financeInfo.financeIncome}}</uni-td>
                    </uni-tr>
                    <uni-tr>
                        <uni-td width="100">描述</uni-td>
                        <uni-td>{{financeInfo.financeDec}}</uni-td>
                    </uni-tr>
                    <uni-tr>
                        <uni-td width="100">状态</uni-td>
                        <uni-td>
                            <view style="width: 100rpx;">
                                <u-text  v-if="financeInfo.financeFlag == 0" size="mini" text="结清"
                                    type="success"></u-text>
                                <u-text  v-if="financeInfo.financeFlag == 1" size="mini"
                                    text="未结" type="primary"></u-text>
                            </view>
                        </uni-td>
                    </uni-tr>
                    <uni-tr>
                        <uni-td width="100">所属表格</uni-td>
                        <uni-td>{{financeInfo.sysType.typeName}}</uni-td>
                    </uni-tr>
                    <uni-tr>
                        <uni-td  width="100">所属类型</uni-td>
                        <uni-td>
                            <view style="width: 100rpx;">
                                <u-tag  v-if="financeInfo.financeIncome != null" size="mini" text="收入"
                                    type="success"></u-tag>
                                <u-tag  v-if="financeInfo.financeExpenditure != null" size="mini"
                                    text="支出"></u-tag>
                            </view>
                            
                        </uni-td>
                    </uni-tr>
                    <uni-tr>
                        <uni-td width="100">创建时间</uni-td>
                        <uni-td>
                            <uni-dateformat v-if="financeInfo.financeCreate!=null" :date="financeInfo.financeCreate"
                                format="yyyy-MM-dd"></uni-dateformat>
                        </uni-td>
                    </uni-tr>
                    <uni-tr v-for="(value, key) in financeBasicData" :key="key">
                        <uni-td width="100">{{ key }}</uni-td>
                        <uni-td>{{ value }}</uni-td>
                    </uni-tr>
                </uni-table>
            </uni-card>
            <uni-card title="结账记录">
                <view v-if="recordList.length>0">
                    <timelineItem class="record-info" v-for="(item,index) in recordList">
                        <view class="tripItem">
                            <view class="title"> {{formatDate(item.recordTime)}}</view>
                            <view class="content">{{finaceType}} {{item.recordMoney}}元</view>
                        </view>
                    </timelineItem>
                </view>
                <view v-else>
                    <u-empty iconSize="200" text="数据为空~" icon="/static/images/empty/data.png"></u-empty>
                </view>
            </uni-card>
        </view>
    </view>
</template>

<script>
    import {
        getFinanceRecordInfo
    } from "@/api/system/record";
    import timeline from '@/components/chenbin-timeline/timeLine.vue'
    import timelineItem from '@/components/chenbin-timeline/timelineItem.vue'
    export default {
        components: {
            timeline,
            timelineItem
        },
        data() {
            return {
                financeBasicData: {},
                financeIds: 0,
                finaceType:"收入",
                financeInfo: {
                    sysType: {}
                },
                recordList: [],
            }
        },
        onLoad(e) {
            this.financeIds = e.financeIds;
            this.init();
        },
        methods: {
            init() {
                this.getFinanceRecordInfo();
            },
            getFinanceRecordInfo() {
                let query = {}
                query.financeIds = this.financeIds
                getFinanceRecordInfo(query).then(res => {
                    this.financeInfo = res.sysFinance
                    this.recordList = res.recordList
                    if(this.financeInfo.financeIncome != null){
                        this.finaceType = "收入"
                    }
                    if(this.financeInfo.financeExpenditure != null){
                        this.finaceType = "支出"
                    }
                     this.financeBasicData = JSON.parse(res.sysFinance.financeInfo);
                })
            },
            formatDate(value) {
                // 假设value的值为 "2023-11-09"
                let date = new Date(value);

                let formattedDate =
                    `${date.getFullYear()}年${date.getMonth() + 1}月${date.getDate()}日`;
                return formattedDate;
            },
            getWeekDay(date) {
                let weekdays = ["星期日", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"];
                return weekdays[date.getDay()];
            },
        }
    }
</script>

<style scoped>
    .title{
        font-size: 15px;
        font-weight: bold;
        color: #ababab;
    }
    .content{
        font-size: 15px;
        color: #000000;
    }
</style>