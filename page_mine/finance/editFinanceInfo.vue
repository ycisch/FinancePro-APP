<template>
    <view>
        <view>
            <u-form :model="form" ref="uForm" :rules="rules">
                <u-form-item label="金额">
                    <u--input border="bottom" placeholder="请输入金额" v-model="form.money"></u--input>
                </u-form-item>
                <u-form-item prop="type" borderBottom @click="clickType" label="分类">
                    <u--input v-model="form.type" disabled disabledColor="#ffffff" placeholder="请选择所属表格"
                        border="none"></u--input>
                    <u-icon slot="right" name="arrow-right"></u-icon>
                </u-form-item>
                <u-form-item prop="date" borderBottom @click="clickCalendar" label="时间">
                    <u--input v-model="form.date" disabled disabledColor="#ffffff" placeholder="请选择时间"
                        border="none"></u--input>
                    <u-icon slot="right" name="arrow-right"></u-icon>
                </u-form-item>
                <u-form-item label="状态">
                    <u-radio-group v-model="form.flag" placement="row" @change="radioGroupChange">
                        <u-radio :customStyle="{marginBottom: '0rpx',marginLeft:'8rpx'}"
                            v-for="(item, index) in radioList" :key="index" :label="item.name" :name="item.id"
                            @change="radioChange">
                        </u-radio>
                    </u-radio-group>
                </u-form-item>
                <u-form-item label="备注">
                    <u--textarea v-model="form.info" placeholder="请输入内容" autoHeight></u--textarea>
                </u-form-item>
                <u-form-item v-for="(value, key) in financeBasicData" :key="key" :label="key">
                    <u-input v-model="financeBasicData[key]" placeholder="请输入金额"></u-input>
                </u-form-item>

                <uni-card title="结账记录">
                    <view v-if="recordList.length>0">
                        <timelineItem @click="timeClick(item)" class="record-info" v-for="(item,index) in recordList">
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

            </u-form>
            <u-button type="primary" text="修改" customStyle="margin-top: 50rpx" @click="submit"></u-button>
            <u-picker :defaultIndex="[0]" :show="showType" :columns="computedTypes" @cancel="typeCancel"
                @close="typeCancel" @confirm="typeConfirm" closeOnClickOverlay @change="typeChange"
                keyName="text"></u-picker>
            <u-datetime-picker ref="picker" :value="form.date" :show="showCalendar" mode="date" closeOnClickOverlay
                @confirm="confirm" @cancel="close" @close="close"></u-datetime-picker>
        </view>
    </view>
</template>

<script>
    import {
        updateFinance
    } from "@/api/system/finance";
    import {
        getFinanceRecordInfo
    } from "@/api/system/record";
    import {
        getTypeList
    } from "@/api/system/redis";
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
                finaceType: "收入",
                financeInfo: {
                    sysType: {}
                },
                recordList: [],
                form: {
                    flag: 0,
                    typeId: null,
                },
                radioList: [{
                        name: '结清',
                        id: "0",
                    },
                    {
                        name: '未结',
                        id: "1",
                    }
                ],
                showCalendar: false,
                showType: false,
                types: [],
                rules: {
                    date: {
                        type: 'string',
                        min: 2,
                        required: true,
                        message: '请选择时间',
                        trigger: ['change']
                    },
                    type: {
                        type: 'string',
                        min: 1,
                        required: true,
                        message: '请选择所属表格',
                        trigger: ['blur', 'change']
                    },
                },
            }
        },
        onReady() {
            // 如果需要兼容微信小程序，并且校验规则中含有方法等，只能通过setRules方法设置规则
            this.$refs.uForm.setRules(this.rules)
        },
        onLoad(e) {
            this.financeIds = e.financeIds;
            this.init();
        },
        computed: {
            computedTypes() {
                return [this.types];
            }
        },
        methods: {
            timeClick(e) {
                console.log(e)
                 const serializedData = encodeURIComponent(JSON.stringify(e));
                uni.navigateTo({
                    // url: '/page_mine/finance/editFinanceRecordInfo?financeIds=' + this.financeIds
                    url: '/page_mine/finance/editFinanceRecordInfo?financeData=' + serializedData
                })
            },
            clickType() {
                this.showType = true;
            },
            clickCalendar() {
                this.showCalendar = true;
                if (this.form.date != undefined && this.form.date != null) {
                    this.$refs.picker.innerValue = new Date(this.form.date)
                }
            },
            typeCancel(e) {
                this.showType = false

            },
            typeChange(e) {},
            typeConfirm(e) {
                if (e.value[0].typeFlag == 1 && this.tabIndex == 1) {
                    this.flag = true
                } else {
                    this.flag = false
                }
                this.form.type = e.value[0].typeName
                this.form.typeId = e.value[0].typeId
                this.infoTypeId = e.value[0].typeId
                this.showType = false
            },
            confirm(e) {
                this.showCalendar = false
                this.form.date = uni.$u.timeFormat(e.value, 'yyyy-mm-dd')
            },
            close() {
                this.showCalendar = false
            },
            // 选中某个单选框时，由radio时触发
            radioChange(e) {
                // console.log(e);
            },
            // 选中任一radio时，由radio-group触发
            radioGroupChange(e) {
                this.form.flag = e
            },
            init() {
                this.getFinanceRecordInfo();
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
            getFinanceRecordInfo() {
                let query = {}
                query.financeIds = this.financeIds
                getFinanceRecordInfo(query).then(res => {
                    this.financeInfo = res.sysFinance
                    this.recordList = res.recordList
                    if (this.financeInfo.financeIncome != null) {
                        this.finaceType = "收入"
                    }
                    if (this.financeInfo.financeExpenditure != null) {
                        this.finaceType = "支出"
                    }
                    this.financeBasicData = JSON.parse(res.sysFinance.financeInfo);
                    this.form.money = this.financeInfo.financeMode == '0' ? this.financeInfo.financeIncome :
                        this.financeInfo.financeExpenditure
                    this.form.date = this.financeInfo.financeCreate
                    this.form.info = this.financeInfo.financeDec
                    this.form.flag = this.financeInfo.financeFlag
                    this.form.type = this.financeInfo.sysType.typeName
                    this.form.typeId = this.financeInfo.sysType.typeId
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
            submit() {
                //提交数据到后台
                this.$refs.uForm.validate().then(res => {
                    uni.$u.toast('校验通过')
                    let sysFinance = {}
                    let strCustomFields = JSON.stringify(this.financeBasicData);
                    if (this.financeInfo.financeMode == 0) {
                        sysFinance.financeIncome = this.form.money
                        sysFinance.financeMode = '0'
                    } else if (this.financeInfo.financeMode == 1) {
                        sysFinance.financeExpenditure = this.form.money
                        sysFinance.financeMode = '1'
                    }
                    sysFinance.financeCreate = this.form.date
                    sysFinance.financeFlag = this.form.flag
                    sysFinance.financeType = this.form.typeId
                    sysFinance.financeDec = this.form.info
                    sysFinance.financeInfo = strCustomFields
                    sysFinance.financeId = this.financeIds
                    console.log(this.form)
                    console.log(strCustomFields)
                    console.log(sysFinance)
                    updateFinance(sysFinance).then(response => {
                        if (response.code == '500') {
                            uni.$u.toast('输入金额有误 请仔细检查')
                            return
                        } else {
                            uni.$u.toast('修改成功')
                            this.getFinanceRecordInfo()
                        }
                    })
                }).catch(errors => {
                    console.log(errors)
                    uni.$u.toast('校验失败')
                })
            },
        }
    }
</script>

<style scoped>
    .title {
        font-size: 15px;
        font-weight: bold;
        color: #ababab;
    }

    .content {
        font-size: 15px;
        color: #000000;
    }
</style>