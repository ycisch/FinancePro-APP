<template>
    <view class="content">
        <view class="top_box">{{ title }}</view>

        <view class="btn_cube" @click="tableToExcel">导出一个表来看</view>

        <view class="tip">tips：合并什么的可以直接用table标签相关的行内属性合并，如colspan、rowspan</view>
        <view class="tip">tips：创建目录时，一个大目录，下面再有一级年月的目录，方便到时候读取目录</view>
        <view class="tip">{{ successTip }}</view>
    </view>
</template>

<script>
    export default {
        data() {
            return {
                title: 'app端导出excel',
                successTip: ''
            };
        },
        methods: {
            tableToExcel() {
                // 要导出的json数据
                const jsonData = [{
                    name: '测试数据',
                    phone: '123456',
                    email: '123@456.com'
                }];

                // 列标题
                let worksheet = 'sheet1';
                let str = '<tr><td>姓名</td><td>电话</td><td>邮箱</td></tr>';

                // 循环遍历，每行加入tr标签，每个单元格加td标签
                for (let i = 0; i < jsonData.length; i++) {
                    str += '<tr>';
                    for (let item in jsonData[i]) {
                        // 增加\t为了不让表格显示科学计数法或者其他格式
                        str += `<td>${jsonData[i][item] + '\t'}</td>`;
                    }
                    str += '</tr>';
                }

                // 下载的表格模板数据
                let template = `<html xmlns:o="urn:schemas-microsoft-com:office:office" 
                xmlns:x="urn:schemas-microsoft-com:office:excel" 
                xmlns="http://www.w3.org/TR/REC-html40">
                <head><!--[if gte mso 9]><xml encoding="UTF-8"><x:ExcelWorkbook><x:ExcelWorksheets><x:ExcelWorksheet>
                <x:Name>${worksheet}</x:Name>
                <x:WorksheetOptions><x:DisplayGridlines/></x:WorksheetOptions></x:ExcelWorksheet>
                </x:ExcelWorksheets></x:ExcelWorkbook></xml><![endif]-->
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
                </head><body><table>${str}</table></body></html>`;

                // 下载模板
                this.exportFile(template);
            },
            exportFile(fileData, documentName = '项目Excel文件') {
                // 创建根目录前先检查目录是否存在
                uni.getFileSystemManager().access({
                    path: `${uni.env.USER_DATA_PATH}/${documentName}`,
                    success: () => {
                        // 根目录已存在，继续创建年月目录和文件
                        this.createFile(fileData,documentName);
                    },
                    fail: () => {
                        // 根目录不存在，创建根目录
                        uni.getFileSystemManager().mkdir({
                            dirPath: `${uni.env.USER_DATA_PATH}/${documentName}`,
                            recursive: true,
                            success: () => {
                                // 根目录创建成功，继续创建年月目录和文件
                                this.createFile(fileData,documentName);
                            },
                            fail: (mkdirErr) => {
                                console.error('创建根目录失败：', mkdirErr);
                            }
                        });
                    }
                });
            },
            createFile(fileData,documentName) {
                // 文件路径
                const filePath = `${uni.env.USER_DATA_PATH}/${documentName}/${this.getFileName()}.xls`;

                // 写入文件
                uni.getFileSystemManager().writeFile({
                    filePath,
                    data: fileData,
                    encoding: 'utf8',
                    success: () => {
                        // 打开文档
                        uni.openDocument({
                            filePath,
                            showMenu: true,
                            success: () => {
                                uni.showToast({
                                    title: '成功导出',
                                    icon: 'success'
                                });
                                this.successTip = `文件位置：${filePath}`;
                            },
                            fail: (openErr) => {
                                console.error('打开文件失败：', openErr);
                            }
                        });
                    },
                    fail: (writeErr) => {
                        console.error('写入文件失败：', writeErr);
                    }
                });
            },
            getYearMonth() {
                let date = new Date();
                let year = date.getFullYear();
                let month = date.getMonth() + 1;
                return `${year}年${month}月`;
            },
            getFileName() {
                return 'excel' + new Date().getTime();
            }
        }
    };
</script>