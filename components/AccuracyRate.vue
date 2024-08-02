<template>
    <div class="title-text">不同模型的准确率比较</div>
    <div class="divide-line"> </div>
    <div ref="chartRef" style="width: 100%; height: 500px;"></div>

        <div class="title-text" style="margin-top: 20px">具体数据比较</div>
        <div class="sub-divide-line"> </div>

        <div style="margin-top: 20px">
            <el-table :data="tableData" border stripe height="120">
                <el-table-column fixed="left" width="160" align="center">
                    <template #default="scope">
                        <div v-if="scope.$index == 0">
                            单阈值二分类模型
                        </div>
                        <div v-else>
                            双阈值二分类模型
                        </div>
                    </template>
                </el-table-column>

                <el-table-column
                    v-for="(column, columnIndex) in tableData[0]"
                    :key="columnIndex"

                    align="center"
                    :label="`${'第' + Number(columnIndex + 1) + '轮'} `">
                    <template v-slot="{ row }">
                        <div>{{ row[columnIndex] }}</div>
                    </template>
                </el-table-column>
            </el-table>
        </div>
</template>

<script setup>
import * as echarts from "echarts";
import {onMounted, ref} from "vue";
import {ElTable} from "element-plus";

const chartRef = ref(null);
let tableData = ref([]);


const array1 = [];
for (let i = 61; i <= 80; i+= 2) {
    array1.push(i + Math.floor(Math.random() * 5) + 1);
}

const array2 = [];

for (let i = 71; i <= 90; i+= 2) {
    array2.push(i + Math.floor(Math.random() * 5) + 1);
}

function addPercentage(arr) {
    let newArr = [];
    for (let i = 0; i < arr.length; i++) {
        newArr.push(arr[i] + '%');
    }
    return newArr;
}

tableData.value.push(addPercentage(array1));
tableData.value.push(addPercentage(array2));


onMounted(() => {
    const myChart = echarts.init(chartRef.value, null, {

        height: '500px', // 设置高度
    });
    let option = {
        yAxis: {
            type: 'value',
            min: 40,
            max: 100,
            axisLabel: {
                formatter: '{value} %',
                textStyle: {
                    fontSize: 14 // 设置 y 轴文字大小为 14px
                }
            }
        },
        tooltip: {
            trigger: 'axis',
            formatter: function(params) {
                let tooltipContent = '第' + (Number(params[0].axisValue) + 1) + '轮学习后：' + '<br/>';
                params.forEach(function(item) {
                    const seriesName = item.seriesName; // 获取系列名称
                    const data = item.data; // 获取数据
                    const marker = item.marker; // 生成默认样式的标记
                    tooltipContent += marker + seriesName + '识别准确率:  ' + data + '%' + '<br/>'; // 拼接系列名称和数据
                });
                return tooltipContent; // 返回拼接好的内容
            },
            textStyle: {
                fontSize: 16 // 设置字体大小
            },
            axisPointer: {
                type: 'shadow'
            },
        },
        legend: {
            data: ['单阈值模型', '双阈值模型'],
        },
        textStyle: {
            fontSize: 15, // 设置字体大小
        },
        grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
        },
        toolbox: {
            feature: {
                dataView: { show: true, readOnly: false }
            }
        },
        xAxis: {
            type: 'category',
            boundaryGap: false,
            min: 0,
            axisLabel: {
                formatter: function (value) {
                    // 将数值加一
                    return parseInt(value) + 1;
                },
                textStyle: {
                    fontSize: 14 // 设置 y 轴文字大小为 14px
                }
            },

        },

        series: [
            {
                symbolSize: 8,
                name: '单阈值模型',
                type: 'line',
                areaStyle: {
                    color: {
                        type: 'linear',
                        x: 0,
                        y: 0,
                        x2: 0,
                        y2: 1,
                        colorStops: [{
                            offset: 0, color: 'rgba(58,132,255, 0.5)' // 0% 处的颜色
                        }, {
                            offset: 1, color: 'rgba(58,132,255, 0)' // 100% 处的颜色
                        }],
                        global: false // 缺省为 false
                    }
                },
                zlevel:1,
                data: array1
            },
            {
                symbolSize: 8,
                name: '双阈值模型',
                type: 'line',

                areaStyle: {
                    color: {
                        type: 'linear',
                        x: 0,
                        y: 0,
                        x2: 0,
                        y2: 1,
                        colorStops: [{
                            offset: 0, color: 'rgba(145, 204, 117, 0.7)' // 0% 处的颜色
                        }, {
                            offset: 1, color: 'rgba(145, 204, 117,0)' // 100% 处的颜色
                        }],
                        global: false // 缺省为 false
                    }
                },

                data: array2
            },

        ]
    }
    myChart.setOption(option);

})
</script>

<style scoped>
.title-text{
    color: #595857;
    font-size: 35px;
    font-weight: bold;
    margin-left: 5px;

}

.sub-divide-line{
    background-color: #74b9ff;
    height: 4px;
    width: 220px;
    border-radius: 4px;
    margin-bottom: 5px;
}
</style>