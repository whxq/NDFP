<template>
    <div class="title-container">
        <el-row>
            <div>
                <el-dropdown placement="bottom-start" class="custom-dropdown">
                    <div class="first-title-text" style="margin-bottom: 5px">
                        双阈值变化(10轮学习): {{curTitle + ' Samples'}}
                    </div>
                    <template #dropdown>
                        <el-dropdown-menu class="custom-dropdown-menu">
                            <el-dropdown-item :class="[activeNum == 1 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('UNSW-NB15',1)">UNSW-NB15 Samples</el-dropdown-item>
                            <el-dropdown-item :class="[activeNum == 2 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('CSE-CIC-IDS-2018',2)">CSE-CIC-IDS-2018 Samples</el-dropdown-item>
                            <el-dropdown-item :class="[activeNum == 3 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('Bot-IoT',3)">Bot-IoT Samples</el-dropdown-item>
                            <el-dropdown-item :class="[activeNum == 4 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('ToN-IoT',4)">ToN-IoT Samples</el-dropdown-item>
                        </el-dropdown-menu>
                    </template>
                </el-dropdown>
                <!--                <div class="divide-line"> </div>-->
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
    <div ref="chartRef" style="width: 100%; margin-top: 30px"></div>


</template>

<script setup>
import * as echarts from "echarts";
import {onMounted, ref, watch} from "vue";
import {AXIS_LABEL_FONT, AXIS_LABEL_NAME_FONT, DATA, DATA_2, TRAINING_DATA} from "../const/data.js";

const chartRef = ref(null);
let tableData = ref([]);
let curTitle = ref('请选择')
let activeNum = ref(-1)
function setTitle(newTitle, newActiveNum){
    activeNum.value = newActiveNum

    TRAINING_DATA.forEach((item) => {
        if (item.name === newTitle) {
            data = item.data
        }
    })

    curTitle.value = newTitle
}

let data = null

let goodThresholds = []
let attackThresholds = []

let attackYAxis = {
    max: 0,
    min: Infinity,
    sum: 0
}
let goodYAxis = {
    max: 0,
    min: Infinity,
    sum: 0
}
let pointAttr = {
    max:0,
    min:Infinity,
    goodMin: Infinity,
    goodMax: 0,
    attackMin: Infinity,
    attackMax: 0,
}
let pointSet = {
    attackTopArea: [],
    attackMiddleArea: [],
    attackBottomArea: [],
    goodTopArea: [],
    goodMiddleArea: [],
    goodBottomArea: [],
}

let outerIndex, gap, goodGap, attackGap, totalAverage, goodFigureHeight, attackFigureHeight, option

//重置参数
function resetAttributes() {
    goodThresholds.length = 0;
    attackThresholds.length = 0;

    attackYAxis.max = 0;
    attackYAxis.min = Infinity;
    attackYAxis.sum = 0;

    goodYAxis.max = 0;
    goodYAxis.min = Infinity;
    goodYAxis.sum = 0;

    outerIndex = 0;

    pointAttr.max = 0;
    pointAttr.min = Infinity;
    pointAttr.goodMin = Infinity;
    pointAttr.goodMax = 0;
    pointAttr.attackMin = Infinity;
    pointAttr.attackMax = 0;

    for (let key in pointSet) {
        if (Array.isArray(pointSet[key])) {
            pointSet[key].length = 0;
        }
    }
}


function iniThresholds(){
    //装填参数
    data.forEach((item,index) => {
        attackYAxis.sum += item.good_threshold
        goodYAxis.sum += 40 - item.attack_threshold
        // 添加索引属性
        item.index = outerIndex++;
        // 将数据压入数组
        goodThresholds.push({ originalValue: item.good_threshold, index: item.index, value: 0 });
        attackThresholds.push({ originalValue: 40 - item.attack_threshold, index: item.index, value: 0 });
    })
}
function regulateGoodData(){
    for (const item of goodThresholds) {
        const difference = item.originalValue;
        if (difference > goodYAxis.max) {
            goodYAxis.max = difference;
        }
        if (difference < goodYAxis.min) {
            goodYAxis.min = difference;
        }
    }
}

function regulateAttackData(){
    for (const item of attackThresholds) {
        const difference = Math.abs(item.originalValue - 40);
        if (difference > attackYAxis.max) {
            attackYAxis.max = difference;
        }
        if (difference < attackYAxis.min) {
            attackYAxis.min = difference;
        }
    }
}

//阶梯量化
function calculateWeight(data, baseValue) {
    // 初始化基准值
    let currentBase = baseValue;
    let tmpData = Array.from(data)
    tmpData.sort((a, b) => a.originalValue - b.originalValue);
    let preValue = tmpData.shift()
    preValue.value = currentBase
    // 遍历数组，计算权值
    tmpData.forEach((item, index) =>{
        let differ = Math.abs(item.originalValue - preValue.originalValue)
        if (differ < 0.01){
            currentBase += 1
        } else if (differ < 0.5){
            currentBase += 5
        } else {
            currentBase += 10
        }

        data.forEach(innerItem => {
            //修改原始数组中的权值
            if (innerItem.originalValue == item.originalValue) {
                innerItem.value = currentBase
            }
        })
        //更新前驱数据值
        preValue = item
    })
    let figureBase = currentBase * 1
    data.forEach(innerItem => {
        //修改原始数组中的权值
        innerItem.value += figureBase
    })
    return currentBase += figureBase
}


function iniPointSet(){
    data?.forEach((item,index) => {
        let startIndex = index * 11
        //本轮的阈值平均值
        let average = (item.attack_threshold + item.good_threshold) / 2
        item.loss.forEach((item, innerIndex)=>{
            let tmpPoint = [startIndex + innerIndex + 1, item]

            if (item < goodYAxis.min) {
                //对应goodBottomArea
                pointSet.goodBottomArea.push(tmpPoint)
                if (item < pointAttr.goodMin){
                    pointAttr.goodMin = item
                }
            } else if (item < goodYAxis.max) {
                //对应goodMiddleArea
                pointSet.goodMiddleArea.push(tmpPoint)
            } else if (item < average) {
                if (item > pointAttr.goodMax){
                    pointAttr.goodMax = item
                }
                //对应goodTopArea
                pointSet.goodTopArea.push(tmpPoint)
            } else if (item < attackYAxis.min) {
                //对应attackBottomArea
                pointSet.attackBottomArea.push(tmpPoint)
            } else if (item < attackYAxis.max) {
                //对应attackMiddleArea
                pointSet.attackMiddleArea.push(tmpPoint)
            } else {
                if (item > pointAttr.attackMax){
                    pointAttr.attackMax = item
                }
                //对应attackTopArea
                pointSet.attackTopArea.push(tmpPoint)
            }

            if(item > pointAttr.max){
                pointAttr.max = item
            }
            if(item < pointAttr.min){
                pointAttr.min = item
            }
        })
    })
}



function iniData(){
    resetAttributes()
    iniThresholds()
    regulateAttackData()
    regulateGoodData()
    iniPointSet()
    gap = attackYAxis.min - goodYAxis.max
    goodGap = goodYAxis.max - goodYAxis.min
    attackGap = attackYAxis.max - attackYAxis.min
    totalAverage = (attackYAxis.min + goodYAxis.max)/2
    goodFigureHeight = calculateWeight(goodThresholds, 0);
    attackFigureHeight = calculateWeight(attackThresholds, 0);
    pointAttr.min = goodYAxis.max - gap
    pointAttr.max = attackYAxis.min + gap
    option = {
        // 指定图表的类型为柱状图
        type: 'bar',
        // 指定图表的标题
        tooltip: {
            trigger: 'item',
            axisPointer: {
                type: 'shadow'
            },
            textStyle: {
                fontSize: 16 // 设置字体大小
            },
            formatter: function(params) {
                let tooltipHtml = ''
                tooltipHtml += params.marker
                tooltipHtml += params.seriesName + '<br\>'
                tooltipHtml += '&nbsp&nbsp Loss: '
                tooltipHtml += '&nbsp' + params.data[1] + '&nbsp'
                return tooltipHtml; // 返回自定义的 tooltip HTML
            }
        },
        textStyle: {
            fontSize: 15, // 设置文字大小
            fontWeight: 400 // 设置文字粗细，可选值包括：'normal', 'bold', 'bolder', 'lighter'
        },
        legend: {

            data: [
                { name: '恶意数据阈值', icon: 'roundRect', itemStyle: { color: 'rgba(240, 86, 84, 0.8)' } },
                { name: '正常数据阈值', icon: 'roundRect', itemStyle: { color: 'rgba(145, 204, 117, 0.8)' }  },
                { name: '恶意数据包', icon: 'roundRect'},
                { name: '正常数据包', icon: 'roundRect'},
            ]
            ,itemGap: 20
        },
        //上下左右距离
        grid: {
            left: '1%',
            right: '2%',
            bottom: '5%',
            top: '7%',
            containLabel: true
        },
        // 指定图表的 X 轴数据
        xAxis: [
            {
                //核心轴
                name: 'FL-epoch',
                nameLocation: 'end', // 设置名称位置为中间
                type: 'category',
                nameTextStyle: {
                    ...AXIS_LABEL_NAME_FONT, // 使用对象扩展合并全局字体样式
                    padding: [20, 0, 0, -5] // 其他名称样式属性
                },
                max:9,
                data: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9],
                axisLabel: {
                    formatter: function (value) {
                        return parseInt(value)+ 1; //
                    },
                    align: 'center',
                    textStyle: {
                        fontSize: 15, // 设置文字大小
                        fontWeight: 550 // 设置文字粗细，可选值包括：'normal', 'bold', 'bolder', 'lighter'
                    },

                }

            },
            {
                type: 'category',
                max:9,
                axisLabel: {
                    show: false  // 隐藏 x 轴上的数字
                },
                show: false
            },
            {
                type: 'value',
                interval:11,
                axisLabel: {
                    show: false  // 隐藏 x 轴上的数字
                },
                // show: false,
                min:0,
                max:110,

            },

        ],
        yAxis: [
            //唯一显示的y轴,实现非均匀的刻度 ->0
            {
                name: 'Loss',
                nameTextStyle: AXIS_LABEL_NAME_FONT,
                type: 'value',
                min:0,
                max:30,

                axisLabel: {
                    textStyle: AXIS_LABEL_FONT,
                    formatter: function (value, index) {
                        if (index == 0 || index == 1 || index == 5 || index == 6){
                            return null
                        } else if (index == 2){
                            return goodYAxis.max.toFixed(1)
                        } else if (index == 4){
                            return attackYAxis.min.toFixed(1)
                        } else if (index == 3){
                            return ((attackYAxis.min + goodYAxis.max)/2).toFixed(1)
                        }
                        return (value);
                    }
                },



            },

            //攻击阈值虚拟Y轴 ->1
            {
                type: 'value',
                min: pointAttr.min,
                max: pointAttr.max,
                axisLabel: {
                    show: false  // 隐藏 x 轴上的数字
                },
                show: false,
            },

            //正常阈值虚拟Y轴 ->2
            {
                type: 'value',
                min:goodYAxis.min,
                max:goodYAxis.max,
                axisLabel: {
                    show: false  // 隐藏 x 轴上的数字
                },
                show: false,
            },

            //正常数据缩放虚拟y轴 ->3
            {
                type: 'value',
                min:0,
                max:goodFigureHeight*3,
                interval:5,
                axisLabel: {
                    textStyle: {
                        fontSize: 14 // 设置 x 轴文字大小为 14px
                    },
                    formatter: function (value, index) {
                        // value 表示原始数据，index 表示数据索引
                        // 这里可以根据需要自定义返回的文本内容
                        return (value);
                    }
                },
                show: false,
            },

            //异常数据缩放虚拟y轴 ->4
            {
                type: 'value',
                min:0,
                max:attackFigureHeight*3,
                inverse: true, // 设置 Y 轴倒置
                interval:5,
                axisLabel: {
                    textStyle: {
                        fontSize: 14 // 设置 x 轴文字大小为 14px
                    },
                    formatter: function (value, index) {
                        // value 表示原始数据，index 表示数据索引
                        // 这里可以根据需要自定义返回的文本内容
                        return (value);
                    }
                },
                show: false,
            },

            //goodBottomArea ->5
            {
                type: 'value',
                min: pointAttr.goodMin,
                max: 2*(goodYAxis.max - pointAttr.goodMin) + pointAttr.goodMin,
                axisLabel: {
                    show: false,
                },
                show: false,
            },

            //数据点中央数据区域 ->6
            {
                type: 'value',
                min:goodYAxis.min - 2*goodGap,
                max:goodYAxis.max + 5*goodGap,
                axisLabel: {
                    show: false,
                },
                show: false,
            },

            //渲染goodTopArea ->7
            {
                type: 'value',
                min: totalAverage - 3*(totalAverage - goodYAxis.max),
                max: totalAverage + 3*(totalAverage - goodYAxis.max),
                axisLabel: {
                    show: false,
                },
                show: false,
            },

            //渲染attackBottomArea ->8
            {
                type: 'value',
                min: totalAverage - 3*(totalAverage - goodYAxis.max),
                max: totalAverage + 3*(totalAverage - goodYAxis.max),
                axisLabel: {
                    show: false,
                },
                show: false,
            },

            //渲染attackMiddleArea ->9
            {
                type: 'value',
                min: totalAverage - 3*(totalAverage - goodYAxis.max),
                max: totalAverage + 3*(totalAverage - goodYAxis.max),
                axisLabel: {
                    show: false,
                },
                show: false,
            },

            //渲染attackTopArea ->10
            {
                type: 'value',
                min: pointAttr.attackMax - 2*(pointAttr.attackMax - totalAverage) -0.2,
                max: pointAttr.attackMax + 0.2,
                axisLabel: {
                    show: false,
                },
                show: false,
            },



        ],


        // 指定图表的数据系列
        series: [
            //渲染goodBottomArea
            {
                name: '正常数据包',
                symbolSize: 9,
                data: pointSet.goodBottomArea,
                type: 'scatter',
                symbol: 'roundRect', // 三角形路径
                itemStyle: {
                    color: '#47885e',
                },
                yAxisIndex: 5,
                xAxisIndex: 2
            },
            //渲染goodMiddleArea
            {
                name: '正常数据包',
                symbolSize: 9,
                data: pointSet.goodMiddleArea,
                type: 'scatter',
                symbol: 'roundRect',
                itemStyle: {
                    color: '#47885e',
                },
                yAxisIndex: 6,
                xAxisIndex: 2
            },
            //渲染goodTopArea
            {
                name: '正常数据包',
                symbolSize: 9,
                data: pointSet.goodTopArea,
                type: 'scatter',
                symbol: 'roundRect',
                itemStyle: {
                    color: '#47885e',
                },
                yAxisIndex: 7,
                xAxisIndex: 2
            },
            //渲染attackBottomArea
            {
                name: '恶意数据包',
                symbolSize: 11,
                data: pointSet.attackBottomArea,
                type: 'scatter',
                symbol: "path://M485.003636 203.403636l-360.261818 609.745455c-55.389091 93.090909 11.636364 210.850909 120.087273 210.850909h720.989091c107.985455 0 175.010909-117.76 120.087273-210.850909l-360.261819-609.745455c-54.458182-91.229091-186.647273-91.229091-240.64 0z",
                itemStyle: {
                    color: '#b7282e',
                },
                yAxisIndex: 7,
                xAxisIndex: 2
            },
            //渲染attackMiddleArea
            {
                name: '恶意数据包',
                symbolSize: 11,
                data: pointSet.attackMiddleArea,
                type: 'scatter',
                symbol: 'path://M485.003636 203.403636l-360.261818 609.745455c-55.389091 93.090909 11.636364 210.850909 120.087273 210.850909h720.989091c107.985455 0 175.010909-117.76 120.087273-210.850909l-360.261819-609.745455c-54.458182-91.229091-186.647273-91.229091-240.64 0z',
                itemStyle: {
                    color: '#b7282e',
                },
                yAxisIndex: 7,
                xAxisIndex: 2
            },
            //渲染attackTopArea
            {
                name: '恶意数据包',
                symbolSize: 11,
                data: pointSet.attackTopArea,
                type: 'scatter',
                symbol: 'path://M485.003636 203.403636l-360.261818 609.745455c-55.389091 93.090909 11.636364 210.850909 120.087273 210.850909h720.989091c107.985455 0 175.010909-117.76 120.087273-210.850909l-360.261819-609.745455c-54.458182-91.229091-186.647273-91.229091-240.64 0z',
                itemStyle: {
                    color: '#b7282e',
                },
                yAxisIndex: 10,
                xAxisIndex: 2
            },

            {
                name: '正常数据阈值',
                type: 'bar',
                barCategoryGap:'0%',
                data: goodThresholds,
                itemStyle: {
                    // 使用渐变填充
                    color: linearGradient2
                },
                label: {
                    show: true,
                    position: 'insideBottom',
                    textStyle: {
                        color: 'gray' // 修改标签的颜色为白色
                    },
                    formatter: function(item) {
                        // value.data 是数据值，index 是索引
                        return item.data.originalValue.toFixed(4)
                    }
                },
                // 指定图表的 Y 轴数据
                yAxisIndex: 3,
                xAxisIndex: 0,
                animationEasing:'cubicInOut',
                tooltip: {
                    show: true,  //此处由于前面定义了全局的tooltip,所以这里必须设置为true才能生效
                    trigger: 'item', // 单独设置样式时必须为item才能生效
                    formatter: (params) => {
                        const maker = '<span style="display:inline-block;border-radius:10px;margin-right:4px;width:10px;height:10px;background-color:rgba(145, 204, 117, 1);"></span>'
                        return maker + ' 第' + (params.dataIndex + 1) + '轮 正常阈值 ' + params.data.originalValue;
                    }
                }
            },

            {
                name: '恶意数据阈值',
                type: 'bar',
                barCategoryGap:'0%',
                data: attackThresholds,
                itemStyle: {
                    // 使用渐变填充
                    color: linearGradient
                },
                label: {
                    show: true,
                    position: 'insideTop',
                    textStyle: {
                        color: 'gray' // 修改标签的颜色为白色
                    },
                    formatter: function(params) {
                        // 自定义标签内容，params.value 是数据值
                        return Math.abs(params.data.originalValue - 40).toFixed(4);
                    }
                },
                // 指定图表的 Y 轴数据
                yAxisIndex: 4,
                xAxisIndex: 1,
                animationEasing:'cubicInOut',
                tooltip: {
                    show: true,  //此处由于前面定义了全局的tooltip,所以这里必须设置为true才能生效
                    trigger: 'item', // 单独设置样式时必须为item才能生效
                    formatter: (params) => {
                        const maker = '<span style="display:inline-block;margin-right:4px;border-radius:10px;width:10px;height:10px;background-color:rgba(240, 86, 84, 1);"></span>'
                        return  maker + ' 第' + (params.dataIndex + 1)  + '轮 异常阈值 '  + Math.abs(params.data.originalValue - 40).toFixed(4);
                    }
                },
            }
        ]
    };
}




let colorStops = [{
    offset: 0,
    color: 'rgba(240, 86, 84, 0.00)'  // 渐变起始颜色
}, {
    offset: 1,
    color: 'rgba(238, 102, 102, 0.55)'    // 渐变结束颜色
}];

let colorStops2= [{
    offset: 0,
    color: 'rgba(145, 204, 117, 0.55)'  // 渐变起始颜色
}, {
    offset: 1,
    color: 'rgba(145, 204, 117, 0.00)'    // 渐变结束颜色
}];

// 创建线性渐变对象
let linearGradient = new echarts.graphic.LinearGradient(0, 0, 0, 1, colorStops);
let linearGradient2 = new echarts.graphic.LinearGradient(0, 0, 0, 1, colorStops2);


// 配置图表选项
let myChart = null
onMounted(() => {
    myChart = echarts.init(chartRef.value, null, {
        height: '755px', // 设置高度
    });
    watch(() => curTitle, ()=>{
        if (data != null){
            iniData()
            myChart.setOption(option);
        }
    }, {deep: true, immediate:true})
    setTitle('UNSW-NB15',1)
})




</script>

<style scoped>
:deep(.choice-item){
    font-size: 17px !important;
    color: #595857 !important;
    font-weight: 600;
}
:deep(.choice-item-selected){
    font-size: 18px !important;
    color: #74b9ff !important;
    font-weight: 600;
}

.sub-divide-line{
    background-color: #74b9ff;
    height: 4px;
    width: 220px;
    border-radius: 4px;
    margin-bottom: 5px;
}
.custom-dropdown-menu{
    border-radius: 10px!important;
}


</style>

<style>
:deep(.el-popper__arrow){
    background-color: red!important;
    /*left: 10px !important; !* 调整箭头位置到 start *!*/
}

</style>


<style>
.title-container{
    width: max-content;
}

.first-title-text{
    color: #595857;
    font-size: 35px;
    font-weight: bold;
    padding-right: 3px;
}


.hint-plug{
    background-color: #ee6666;
    height: min-content;
    padding: 3px 6px 3px 6px;
    margin-left: 8px;
    color: #F0F0F0;
    font-weight: bold;
    font-size: 18px;
    border-radius: 10px;
}

</style>