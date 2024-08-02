<template>
    <div class="title-container">
        <el-row>
            <div>
                <el-dropdown placement="bottom-start" class="custom-dropdown">
                    <div class="first-title-text" style="margin-bottom: 5px;">
                        交换机S{{activeNum}}效果评估: {{curTitle + ' Reduced'}}
                    </div>
                    <template #dropdown>
                        <el-dropdown-menu class="custom-dropdown-menu">
                            <el-dropdown-item :class="[activeNum == 1 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('TON-IoT',1)">S1：TON-IoT Reduced</el-dropdown-item>
                            <el-dropdown-item :class="[activeNum == 2 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('CIC-IDS-2018',2)">S2：CIC-IDS-2018 Reduced</el-dropdown-item>
                            <el-dropdown-item :class="[activeNum == 3 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('UNSW-NB15',3)">S3：UNSW-NB15 Reduced</el-dropdown-item>

                        </el-dropdown-menu>
                    </template>
                </el-dropdown>
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

    <div ref="chartRef" style="width: 100%; margin-top: 20px"></div>
    <div style="width: max-content;">
        <div class="title-text" style="margin-right: 5px">整体检测准确率: {{accuracyRate[activeNum]}}</div>
        <div class="sub-divide-line"> </div>
    </div>

    <div style="margin-top: 10px">
        <el-table :data="tableData" :key="activeNum" border stripe  >

            <el-table-column fixed="left" width="100" align="center">
                <template #default="scope">
                    <div style="font-weight: bold!important;">
                        {{val[scope.$index]}}
                    </div>
                </template>
            </el-table-column>


            <el-table-column
                v-for="(column, columnIndex) in tableData[1]"
                :min-width = 140
                align="center"
                :label="`${nameArray[columnIndex]}`">
                <template v-slot="{ row }">
                    <div>{{ row[columnIndex]}}</div>
                </template>
            </el-table-column>

        </el-table>
    </div>
</template>

<script setup>
import * as echarts from "echarts";
import {onMounted, ref} from "vue";
import {ElTable} from "element-plus";
import {ATTACK_STATISTICS, AXIS_LABEL_FONT, AXIS_LABEL_NAME_FONT, TRAINING_DATA} from "../const/data.js";
let myChart

let curTitle = ref('请选择')
let activeNum = ref(-1)
let data = null
let accuracy = null
let val = ['Precision','Recall','F1-Score', 'Support']
//需要展示的参数
let nameArray = ref([])
let precision = ref([])
let recall = ref([])
let f1Score = ref([])
let support = ref([])
let supportRatio = ref([])
let macroAvg  = []
let weightedAvg  = []
let borderRadiusSet = [
    [8, 8, 0, 0],
    [8, 8, 0, 0],
    [5, 5, 0, 0],
    [5, 5, 0, 0],
    [5, 5, 0, 0],
]
let accuracyRate = ['','98.1%','98.7%','97.3%']


let divideRage = [85,75,65,55]
const chartRef = ref(null);
let tableData = ref([]);
let curSwitch = ref('')

function setTitle(newTitle, newActiveNum){
    activeNum.value = newActiveNum
    ATTACK_STATISTICS.forEach((item) => {
        if (item.dataset === newTitle) {
            accuracy = item.accuracy
            data = item.data
        }
    })

    //更新当前数据集
    curTitle.value = newTitle
    updateDataSet()
}
// setTitle('UNSW-NB15', 1)
//更新需要展示的柱状图的数据集
function updateDataSet(){
    let labelStyle
    if (curTitle.value === 'CIC-IDS-2018') {
        labelStyle = { // 设置标签的字体样式
            fontSize: 16, // 字体大小
            fontWeight: 'bold', // 字体粗细
            // color: '#eaeaea'
        }
    } else {
        labelStyle = { // 设置标签的字体样式
            fontSize: 11, // 字体大小
            fontWeight: 'bold', // 字体粗细
            // color: '#eaeaea'
        }
    }

    let totalCount = 0
    nameArray.value = data.map(item => item.class);
    precision.value = data.map(item => item.precision);
    recall.value = data.map(item => item.recall);
    f1Score.value = data.map(item => item['f1-score']);
    // while (tableData.value.length > 0){
    //     tableData.value.pop()
    // }
    tableData.value.length = 0
    console.log('data',tableData.value)


    tableData.value.push(precision.value.map(item => Number(item).toFixed(2)))
    tableData.value.push(recall.value.map(item => Number(item).toFixed(2)))
    tableData.value.push(f1Score.value.map(item => item))
    let newOption = {
        // 指定图表的类型为柱状图
        type: 'bar',
        // 指定图表的标题
        tooltip: {
            trigger: 'axis',
            axisPointer: {
                type: 'shadow'
            },
            textStyle: {
                fontSize: 16 // 设置字体大小
            },
            formatter(params) {
                var relVal = params[0].name;
                let tag = ''
                for (var i = 0, l = params.length; i < l; i++) {
                    // 使用 <span> 标签并通过 CSS 设置右对5
                    if (i <= 1){
                        tag = '%'
                    } else tag = ''
                    relVal += '<br/>' + params[i].marker + params[i].seriesName + ':' + '<span style="float:right; margin-left: 15px">' + Number(params[i].value).toFixed(2) + tag +'</span>';
                }
                return relVal;
            }

        },
        textStyle: {
            fontSize: 15, // 设置文字大小
            fontWeight: 400 // 设置文字粗细，可选值包括：'normal', 'bold', 'bolder', 'lighter'
        },
        legend: {
            data: ['Precision', 'Recall', 'F1-Score','Proportion'],
            itemGap: 20
        },
        grid: {
            left: '4%',
            right: '4%',
            bottom: '10%',
            top:'10%',
        },
        // 指定图表的 X 轴数据
        xAxis: [
            {
                name:'攻击类型',
                nameLocation: 'middle',
                nameTextStyle: {
                    ...AXIS_LABEL_NAME_FONT, // 使用对象扩展合并全局字体样式
                    padding: [0, 0, 0, 1150] // 其他名称样式属性
                },
                type: 'category',
                axisLabel: {
                    // rotate: 20, // 旋转角度
                    interval: 0, // 显示所有标签x
                    textStyle: {
                        fontSize: 14, // 设置文字大小
                        fontWeight: 400 // 设置文字粗细，可选值包括：'normal', 'bold', 'bolder', 'lighter'
                    }
                },
                data: nameArray.value,
            },
        ],
        // 指定图表的 Y 轴数据
        yAxis: [{

            nameTextStyle: {
                ...AXIS_LABEL_NAME_FONT, // 使用对象扩展合并全局字体样式
                padding: [-10, 25, 5, 0] // 其他名称样式属性
            },
            type: 'value',
            axisLabel: {
                // formatter: function (value, index) {
                //     return (value + '%');
                // },
                textStyle: AXIS_LABEL_FONT
            },
            max:1,
            min:0
            },


        ],
        series: [
            {
                name: 'Precision',
                type: 'bar',
                data: precision.value,
                showBackground: true,
                itemStyle: {
                    borderRadius: borderRadiusSet[activeNum.value] // 分别表示左上、右上、右下、左下的圆角半径，可根据需求调整
                },
                label: {
                    show: true,
                    position:'insideTop',
                    formatter: (params) => Number(params.value).toFixed(2),
                    textStyle: labelStyle
                }
            },
            {
                name: 'Recall',
                type: 'bar',
                data: recall.value,
                showBackground: true,
                itemStyle: {
                    borderRadius: borderRadiusSet[activeNum.value] // 分别表示左上、右上、右下、左下的圆角半径，可根据需求调整
                },
                label: {
                    show: true,
                    position:'insideTop',
                    formatter: (params) => Number(params.value).toFixed(2),
                    textStyle: labelStyle
                }
            },
            {
                name: 'F1-Score',
                type: 'bar',
                data: f1Score.value,
                showBackground: true,
                itemStyle: {
                    borderRadius: borderRadiusSet[activeNum.value] // 分别表示左上、右上、右下、左下的圆角半径，可根据需求调整
                },
                label: {
                    show: true,
                    position:'insideTop',
                    formatter: (params) => Number(params.value).toFixed(2),
                    textStyle: labelStyle
                }
            }
        ]
    };
    myChart.setOption(newOption, true)
}


// tableData.value.push(attackData.map(item => item.actualCount))



// 根据准确率获取颜色的函数
function getColorByAccuracy(accuracy) {
    // 根据实际需求定义不同颜色区间和对应颜色
    if (accuracy >= divideRage[0]) {
        return '#91cc75'; // 绿色
    } else if (accuracy >= divideRage[1]) {
        return '#5470c6'; // 黄色
    } else if (accuracy >= divideRage[2]) {
        return '#fac858'; // 黄色
    } else if (accuracy >= divideRage[3]) {
        return '#73c0de'; // 橙色
    } else {
        return '#ee6666'; // 红色
    }
}

onMounted(() => {

    myChart = echarts.init(chartRef.value, null, {height: '565px'});
    setTitle('TON-IoT', 1)

})
</script>

<style scoped>
:deep(.choice-item){
    font-size: 20px !important;
    color: #595857 !important;
    font-weight: 600;
    padding-left: 20px;
    padding-right: 20px;
}
:deep(.choice-item-selected){
    font-size: 20px !important;
    color: #74b9ff !important;
    font-weight: 600;
    padding-left: 20px;
    padding-right: 20px;
}


</style>

<style>
.sub-divide-line{
    background-color: #74b9ff;
    height: 4px;
    width: 100%;
    border-radius: 4px;
    margin-bottom: 5px;
}
:deep(.el-table){
    width:99.9% !important;
}
.el-popper {
    /*left: 305px!important;*/
    width: max-content!important;
}
:deep(.el-table__fixed-body-wrapper) {
    top: 40px !important;
}
:deep(span.el-popper__arrow::before){
    position: absolute;
    left: 100px!important;
}

</style>