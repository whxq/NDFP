<template>
    <div class="title-container">
        <el-row>
            <div>
                <div class="first-title-text" style="line-height: 35px; margin-bottom: 5px">
                    不同重放速率下的准确率和假阳性率
                </div>
                <div class="divide-line"> </div>
            </div>
            <el-popover
                placement="bottom"
                title="说明："
                :width="150"
                trigger="hover"
                content="this is content, this is content, this is content"
            >
                <template #reference>
                    <div class="hint-plug">
                        Historical Data
                    </div>
                </template>
                <template #default>
                    <div style="font-size: 15px; width: 200px">
                        &nbsp;&nbsp;&nbsp;&nbsp;由于模型训练非常耗时，实时数据获取较慢，因此展示历史数据.
                    </div>
                </template>
            </el-popover>
        </el-row>


    </div>

    <div ref="chartRef" style="width: 100%; height: 585px; margin-top: 30px"></div>

    <div class="title-text" style="margin-top: 10px">具体数据</div>
    <div class="sub-divide-line"> </div>

    <div style="margin-top: 10px">
        <el-table :data="tableData" border stripe height="120">
            <el-table-column fixed="left" width="140" align="center">
                <template #default="scope">
                    <div v-if="scope.$index == 0" style="font-weight: bold; font-size: 15px">
                        检测准确率
                    </div>
                    <div v-else style="font-weight: bold; font-size: 15px">
                        假阳性率
                    </div>
                </template>
            </el-table-column>

            <el-table-column
                v-for="(column, columnIndex) in tableData[0]"
                :key="columnIndex"
                align="center"
                :label="`${xData[columnIndex] + 'pps'} `">
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
import {AXIS_LABEL_FONT, AXIS_LABEL_NAME_FONT} from "../const/data.js";

const chartRef = ref(null);
let tableData = ref([]);


const xData = [100, 200, 300, 400, 500, 600, 700, 800, 900, 1000];
const array1 = [98.18, 97.79, 97.79, 96.75, 96.74, 96.21, 95.92, 95.70, 96.71, 96.72];
const array2 = [0.90, 1.04, 1.11, 1.29, 1.40, 1.57, 1.61, 1.76, 1.58, 1.59];


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

        height: '580px', // 设置高度
    });
    let option = {

        yAxis: [
            {
                nameGap:17,
                name: 'Rate 单位%',
                nameTextStyle: {
                    ...AXIS_LABEL_NAME_FONT, // 使用对象扩展合并全局字体样式
                    padding: [-10, 15, 5, 0] // 其他名称样式属性
                },
                type: 'value',
                min: 0,
                max: 100,
                axisLabel: {
                    formatter: function (value, index) {
                        console.log(index)
                        return (value + '%');
                    },
                    textStyle: AXIS_LABEL_FONT
                },
            },
            {
                type: 'value',
                min: 0,
                max: 60,
                axisLabel: {
                    show: false,
                },
                show: false,
            }
        ],
        tooltip: {
            trigger: 'axis',
            // formatter: function(params) {
            //     let tooltipContent = '数据包重放速率' + (Number(params[0].axisValue)) + 'pps：' + '<br/>';
            //     params.forEach(function(item) {
            //         const seriesName = item.seriesName; // 获取系列名称
            //         const data = item.data; // 获取数据
            //         const marker = item.marker; // 生成默认样式的标记
            //         tooltipContent += marker + seriesName + ': ' + '<span style="float:right">' + data + '%' + '<br/>'; // 拼接系列名称和数据
            //     });
            //     return tooltipContent; // 返回拼接好的内容
            // },
            formatter(params) {
                var relVal = params[0].name + 'pps';
                let tag = ''
                for (var i = 0, l = params.length; i < l; i++) {
                    // 使用 <span> 标签并通过 CSS 设置右对5
                    if (i <= 1){
                        tag = '%'
                    } else tag = ''
                    relVal += '<br/>' + params[i].marker + params[i].seriesName + ':' + '<span style="float:right; margin-left: 15px">' + Number(params[i].value).toFixed(2) + tag +'</span>';
                }
                return relVal;
            },
            textStyle: {
                fontSize: 16 // 设置字体大小
            },
            axisPointer: {
                type: 'shadow'
            },
        },
        legend: {
            data: ['检测准确率', '假阳性率'],
            textStyle: {
                fontSize: 15,
                fontWeight: 400
            }
            ,itemGap: 20
        },

        grid: {

            left: '2%',
            right: '4%',
            bottom: '9%',
            containLabel: true
        },
        // toolbox: {
        //     feature: {
        //         dataView: { show: true, readOnly: false }
        //     }
        // },
        xAxis: {
            name: '数据包重放速率/pps',
            // nameTextStyle: AXIS_LABEL_NAME_FONT,
            nameLocation: 'middle',
            nameTextStyle: {
                ...AXIS_LABEL_NAME_FONT, // 使用对象扩展合并全局字体样式
                padding: [20, 0, 0, 967] // 其他名称样式属性
            },
            type: 'category',
            boundaryGap: false,
            min: 0,
            axisLabel: {
                textStyle: AXIS_LABEL_FONT
            },
            data: xData

        },


        series: [
            {
                symbolSize: 8,
                name: '检测准确率',
                type: 'line',
                areaStyle: {
                    color: {
                        type: 'linear',
                        x: 0,
                        y: 0,
                        x2: 0,
                        y2: 1,
                        colorStops: [{
                            offset: 0, color: 'rgba(58,132,255, 0.45)' // 0% 处的颜色
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
                name: '假阳性率',
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
                yAxisIndex:1,
                data: array2
            },

        ]
    }
    myChart.setOption(option);

})
</script>

<style scoped>
.title-container{
    height: min-content;
}

.sub-divide-line{
    background-color: #74b9ff;
    height: 4px;
    width: 150px;
    border-radius: 4px;
    margin-bottom: 5px;
}
</style>