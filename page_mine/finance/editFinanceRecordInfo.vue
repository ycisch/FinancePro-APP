<template>
    <view>
        <u-form :model="form" ref="uForm">
            <u-form-item label="金额">
                <u--input border="bottom" placeholder="请输入金额" v-model="form.recordMoney"></u--input>
            </u-form-item>
            <u-form-item prop="date" borderBottom @click="clickCalendar" label="时间">
                <u--input v-model="form.recordTime" disabled disabledColor="#ffffff" placeholder="请选择时间"
                    border="none"></u--input>
                <u-icon slot="right" name="arrow-right"></u-icon>
            </u-form-item>
            <u-form-item label="备注">
                <u--textarea v-model="form.recordInfo" placeholder="请输入内容" autoHeight></u--textarea>
            </u-form-item>
        </u-form>
        <u-button type="primary" text="修改" customStyle="margin-top: 50rpx" @click="submit"></u-button>
        <u-datetime-picker ref="picker" :value="form.date" :show="showCalendar" mode="date" closeOnClickOverlay
            @confirm="confirm" @cancel="close" @close="close"></u-datetime-picker>
    </view>
</template>

<script>
    import {
        updateRecord
    } from "@/api/system/record";
    export default {
        data() {
            return {
                form: {},
                showCalendar: false
            }
        },
        onLoad(options) {
            const financeDataString = decodeURIComponent(options.financeData);
            const financeData = JSON.parse(financeDataString);
            this.form = financeData

            // 现在，financeData 包含原始的对象数据
            console.log(this.form);
        },
        methods: {
            clickCalendar() {
                this.showCalendar = true;
                if (this.form.recordTime != undefined && this.form.recordTime != null) {
                    this.$refs.picker.innerValue = new Date(this.form.recordTime)
                }
            },
            confirm(e) {
                this.showCalendar = false
                this.form.date = uni.$u.timeFormat(e.value, 'yyyy-mm-dd')
            },
            close() {
                this.showCalendar = false
            },
            submit() {
                let sysRecord = {}
                console.log(this.form)
                updateRecord(this.form).then(response => {
                    if (response.code == '500') {
                        uni.$u.toast('输入金额有误 请仔细检查')
                        return
                    } else {
                        uni.$u.toast('修改成功')
                    }
                })
            },
        }
    }
</script>

<style>
</style>