<template>
    <div  style=" position: relative; height: 100%">
        <div class="title-container">
            <el-row>
                <div class="first-title-text" style="line-height: 35px; margin-bottom: 5px">
                    交换机S1识别出的异常流类型及占比
                </div>
                <div @click="showStdGraph()" class="hint-plug" v-if="isOver" style="padding: 4px 7px 4px 6px">
                    {{ (isCompareMode)?'收起基准数据':'展开基准数据' }}
                </div>
            </el-row>
        </div>
        <div ref="chartRef" style="width: 100%;margin-top: 20px; height: 155px;"></div>

        <div style="width: 100%; height: 220px" v-show="isCompareMode">
            <div class="first-title-text" style="line-height: 35px; margin-bottom: 5px">
                基准数据
            </div>
            <div ref="chartRefStd" v-if="isCompareMode" style="width: 100%; margin-top:10px;height: 155px;"></div>

        </div>

        <el-row v-show="!isCompareMode">
            <transition name="draw">
                <div class="title-container">
                    <div class="first-title-text" style="line-height: 35px; margin-bottom: 7px">
                        具体异常流信息(部分)
                    </div>
                </div>
            </transition>


            <div class="title-container" style="margin-left: 385px">
                <div class="first-title-text" style="line-height: 35px; margin-bottom: 7px">
                    数据面板
                </div>
            </div>
        </el-row>

        <div style="position: absolute; bottom: -5.2%;">
            <el-container style="margin-top: 5px;   transition: height 0.7s ease"
                          :style="{height:(isCompareMode)?'350px':'570px'}">
                <el-aside style="width: 720px;" class="table-container">
                    <el-table
                        ref="myTable"
                        style="transition: height 0.7s ease"
                        :height="(isCompareMode)?'330':'550'"
                        :data="tableData"
                        @cell-mouse-enter="clearScroll"
                        @mouseleave.native="createScroll"
                        stripe
                    >
                        <el-table-column
                            label="Source IP"
                            width="210"
                            align="center">
                            <template #default="scope">
                                <el-row justify="end">
                                    <div>
                                        {{scope.row.srcIP}}
                                    </div>
                                    <el-text class="hint-plug-line" :style="{backgroundColor: '#ee6666'}">
                                        {{'Unknown'}}
                                    </el-text>
                                </el-row>
                            </template>
                        </el-table-column>

                        <el-table-column
                            label="Destination IP"
                            width="130"
                            align="center">
                            <template #default="scope">
                                <el-row justify="center">
                                    <div>
                                        {{scope.row.dstIP}}
                                    </div>
                                    <el-text class="hint-plug-line" :style="{backgroundColor: getHostName(scope.row.dstIP).color}">
                                        {{getHostName(scope.row.dstIP).hostName}}
                                    </el-text>
                                </el-row>
                            </template>
                        </el-table-column>

                        <el-table-column
                            prop="srcPort"
                            label="Source Port"
                            width="110"
                            align="center">
                        </el-table-column>
                        <el-table-column
                            prop="dstPort"
                            label="Destination Port"
                            width="140"
                            align="center">
                        </el-table-column>
                        <el-table-column
                            prop="name"
                            label="Attack Type"
                            align="center">
                        </el-table-column>

                    </el-table>
                </el-aside>


                <el-main class="panel-container" :style="{height:(isCompareMode)?'350px':'570px'}"
                         style="transition: height 0.7s ease">
                    <el-scrollbar>
                        <div ref="innerRef" style="width: 100%; margin-top: 8px"></div>
                        <div ref="innerRef2" style="width: 100%; margin-top: 25px"></div>
                    </el-scrollbar>

                </el-main>
            </el-container>

        </div>
    </div>



</template>

<script setup>
import {nextTick, onMounted, onUnmounted, ref, watch} from 'vue';
import {ElNotification, ElTable} from 'element-plus';
import * as echarts from "echarts";
import {
    AXIS_LABEL_FONT,
    AXIS_LABEL_FONT_S,
    AXIS_LABEL_NAME_FONT, REALTIME_DATA,
    TRAINING_DATA,
    TRAINING_DATA_TYPE
} from "../const/data.js";
const chartRef = ref(null);
const chartRefStd = ref(null);
const innerRef = ref(null);
const innerRef2 = ref(null);
const myTable = ref();
let isCompareMode = ref(false);
let tableData = ref([]);
let count = 0
let curTitle = ref('Switch1')
let activeNum = ref(1)
let attackCount = {
    "Xss": 0,
    "Scanning": 0,
    "DDoS": 0,
    "DoS": 0,
    "injection": 0,
    "password": 0,
    "Backdoor": 0,
    "mitm": 0,
    "ransomware": 0,
}
let timer = null //全局计时器，刷新当前表格
let i = 0
let maxItem = 13
let isOver = ref(false)
let realTimeDataBak = []

//虚拟化表格的列
const columns = [
    { key: 'time', dataKey: 'time', title: '时间', width: 120 },
    { key: 'srcIp', dataKey: 'srcIp', title: '源IP地址', width: 165 },
    { key: 'dstIp', dataKey: 'dstIp', title: '目的IP地址', width: 165 },
    // { key: 'srcMac', dataKey: 'srcMac', title: '源MAC地址', width: 152 },
    // { key: 'dstMac', dataKey: 'dstMac', title: '目的MAC地址', width: 152 },
    { key: 'length', dataKey: 'length', title: '数据包长度', width: 100 },
    { key: 'proto', dataKey: 'proto', title: '协议类型', width: 100 },
    { key: 'sPort', dataKey: 'sPort', title: '源端口号', width: 80 },
    { key: 'dPort', dataKey: 'dPort', title: '目的端口号', width: 90 }
];
function generateRandomObject(thresholds, maxValues) {
    let newObj = {};
    for (const prop in thresholds) {
        if (Math.random() > thresholds[prop] && currentValues[prop] < maxValues[prop]) {
            const randomValue = Math.floor( Math.random()*Math.random()* (maxValues[prop])/4);
            newObj[prop] = randomValue;
            currentValues[prop] += randomValue; // 更新当前值
        }
    }
    return newObj;
}

const currentValues = {
    "Xss": 0,
    "Scanning": 0,
    "DDoS": 0,
    "DoS": 0,
    "injection": 0,
    "password": 0,
    "Backdoor": 0,
    "mitm": 0,
    "ransomware": 0,
}

// 示例用法
const thresholds = {
    "Xss": 0.5,
    "Scanning": 0.7,
    "DDoS": 0.3,
    "DoS": 0.6,
    "injection": 0.4,
    "password": 0.8,
    "Backdoor": 0.2,
    "mitm": 0.9,
    "ransomware": 0.1,
};

const maxValues = {
    "Xss": 48652,
    "DDoS": 40000,
    "password": 22557,
    "Scanning": 21335,
    "injection": 13686,
    "DoS": 12665,
    "Backdoor": 352,
    "mitm": 124,
    "ransomware": 65,
}

const hostStyle = {
    '10.0.1.1': {
        hostName: 'Host1',
        color: '#ee6666'
    },
    '10.0.2.2': {
        hostName: 'Host2',
        color: '#73c1df'
    }
}


let trainingDataType = TRAINING_DATA_TYPE

function setTitle(newTitle, newActiveNum){
    activeNum.value = newActiveNum
    curTitle.value = newTitle
}

let curData = null
let seriesData = [];
let seriesDataStd = [];
let xAxisMax = 0

function getHostName(name){
    return hostStyle[name]
}

function getRandomInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}
function generateRandomIPv4() {
    let ip = [];
    for (let i = 0; i < 4; i++) {
        ip.push(getRandomInt(1, 2)); // 生成0到255之间的随机整数作为IP的一部分
    }
    return ip.join('.'); // 将数组连接成IPv4地址的字符串形式
}
function generateRandomSourceDestIP() {
    let sourceIP = generateRandomIPv4(); // 生成随机的源IP地址
    let destIP = generateRandomIPv4();   // 生成随机的目的IP地址
    return { sourceIP, destIP }; // 返回包含源IP和目的IP的对象
}

//读取数据，更新当前集合
function readRecord(){
    if (realTimeDataBak.length == 0){
        return
    }


    console.log(4578)
    let dataItem = realTimeDataBak.shift()
    //更新数据面板的参数
    updateDataPanel(dataItem.dataPanel.speed, dataItem.dataPanel.countOfPackets)

    //更新表格，参数是表格
    addTableItems(dataItem.packetsInfo)

    //更新主柱状图
    updateMainGraph(dataItem.attackStatistics)
}

let MaxStd = 0
let total = 159426
function iniSeriesData() {
    xAxisMax = 0
    for (let key in maxValues) {
        // key 是属性名，attackCount[key] 是对应的属性值
        let seriesItem = {
            name: key,
            type: 'bar',
            stack: 'total',
            label: {
                offset: [0, 3],
                show: true, // 显示标签
                formatter: function (params) {
                    let percentage = ((maxValues[key]/total)*100).toFixed(2)
                    if (percentage > 5)
                        return  percentage + '%'
                    else
                        return ''
                },
                textStyle: { // 设置标签的字体样式
                    // color: '#F7F7F7', // 字体颜色
                    fontSize: 25, // 字体大小
                    fontWeight: 'bold' // 字体粗细
                }
            },
            emphasis: {
                focus: 'none'
            },
            data: [maxValues[key]]
        };
        MaxStd += maxValues[key]
        seriesDataStd.push(seriesItem);
    }
}

function stimulateCountData(){
    // seriesData.forEach((item) => {
    //     let randomIncrement = getRandomInt1, 10);
    //     if (randomIncrement <= 7){
    //         randomIncrement = 0
    //     }
    //     xAxisMax += randomIncrement
    //     item.data[0] += randomIncrement
    // })

    //表示一轮查询的数据格式，至少需要40条这样的数据添加至一个数组
    // let singleData = {
    //     //packetsInfo每次包含2或者3条
    //     packetsInfo: [{srcIp: '1.1.1.1', dstIp: '1.1.1.1', srcPort: '0', dstPort: '0', attackType: 'Dos'}
    //                  ,{srcIp: '1.1.1.1', dstIp: '1.1.1.1', srcPort: '0', dstPort: '0', attackType: 'Dos'}],
    //     //第一个属性是流量速率，第二个属性是一个时间间隔内捕获的数据包数量
    //     dataPanel: {speed: 100, countOfPackets: 100},
    //     attackStatistics: {
    //         "Xss": 1,
    //         "Scanning": 1,
    //         "DDoS": 1,
    //         //....没检测到不要添加
    //     }
    // }

    updateMainGraph(generateRandomObject(thresholds, maxValues))
    // myChart.setOption(option, true)
}

//如果全部添加到了集合当中说明已经不需要添加了，进入算法case2
let isComplete = false


//更新主图，additionalDataItem是一个包含各种攻击属性的对象，其中存在的属性是需要添加到主图的部分
//1.如果存在这个属性就累加
//2.如果不存在这个属性就创建并添加
function updateMainGraph(additionalDataItem){
    if (!isComplete){
        //遍历待添加元素
        Object.keys(additionalDataItem).forEach((key)=>{
            xAxisMax += additionalDataItem[key]
            let isExist = false
            seriesData.forEach((seriesItem)=>{
                if (seriesItem.name === key) { //表示已经存在这个元素，进行追加
                    isExist = true
                    seriesItem.data[0] +=  additionalDataItem[key]
                }
            })
            if (!isExist && additionalDataItem[key] != 0 ) { //添加不存在的元素
                let seriesItem = {
                    name: key,
                    type: 'bar',
                    stack: 'total',
                    label: {
                        show: false
                    },
                    emphasis: {
                        focus: 'none'
                    },
                    data: [additionalDataItem[key]]
                };
                seriesData.push(seriesItem)

            }
        })

    }
    seriesData.sort((a, b) => b.data[0] - a.data[0])
    //重新绘制图像
    myChart.setOption(option, true)
}

let scrollTimer = null
const clearScroll = () => {
    clearInterval(scrollTimer)
    scrollTimer = null
}

const createScroll = () => {
    clearScroll()
    // 拿到 table
    const table = myTable.value.layout.table.refs
    // 拿到可以滚动的元素
    const tableWrapper = table.bodyWrapper.firstElementChild.firstElementChild

    scrollTimer = setInterval(() => {

        // 判断是否滚动到底部，如果到底部了置为0（可视高度+距离顶部=整个高度）
        if (tableWrapper.clientHeight + tableWrapper.scrollTop == tableWrapper.scrollHeight) {

        } else tableWrapper.scrollTop += 2
    }, 3)
}

const addNewItem = () => {
    for (let j = 0; j < 3; j++) {
        let keys = Object.keys(attackCount);
        // 随机选择一个属性名
        let randomKey = keys[Math.floor(Math.random() * keys.length)]
        let ip = generateRandomSourceDestIP()
        // 添加新内容到表格数据
        const newItem = {
            srcIP: '1.0.0.1',
            dstIP: '1.0.2.2',
            srcPort: getRandomInt(120,1000),
            dstPort: getRandomInt(120,1000),
            name: randomKey
        };
        tableData.value.push(newItem);
    }
};

function addTableItems(packetItems){
    packetItems.forEach((items) => {
        const newItem = {
            srcIP: items.srcIp,
            dstIP: items.dstIp,
            srcPort: items.srcPort,
            dstPort: items.dstPort,
            name:items.attackType
        };
        tableData.value.push(newItem);
    })
}




function cleanAttributes(){
    xAxisMax = 0
    seriesData.length = 0
}

let trigger = true
function showStdGraph(){
    if (trigger) {
        isCompareMode.value = true
        showPercentage()
        nextTick(()=>{
            myChartStd = echarts.init(chartRefStd.value, null, {
                height: '140px', // 设置高度
            });
            myChartStd.setOption(optionStd)
        })
    } else {
        closePercentage()
        isCompareMode.value = false
    }
    trigger = !trigger
}

function showPercentage(){
    seriesData.forEach((item)=>{
        item.label = {
            offset: [0, 3], // 在垂直方向上向上偏移10像素
            show: true, // 显示标签
            formatter: function (params) {
                let percentage = ((item.data[0]/xAxisMax)*100).toFixed(2)
                if (percentage > 5)
                    return  percentage + '%'
                else
                    return ''
            },
            textStyle: { // 设置标签的字体样式
                // color: '#F7F7F7', // 字体颜色
                fontSize: 25, // 字体大小
                fontWeight: 'bold' // 字体粗细
            }
        }
    })
    myChart.setOption(option, true)
}

function closePercentage(){
    seriesData.forEach((item)=>{
        item.label = {
            show: false, // 显示标签
        }
    })
    myChart.setOption(option)
}

onMounted(() => {

})
onUnmounted(() => {
    clearScroll()
    notification?.close()
    clearInterval(timer)
})
//堆叠柱状图
let option = {

    tooltip: {
        trigger: 'axis',
        axisPointer: {
            // Use axis to trigger tooltip
            type: 'shadow' // 'shadow' as default; can also be 'line' or 'shadow'
        },

        position: function (pos, params, el, elRect, size) {
            let posX = pos[0];
            let posY = pos[1];
            let tooltipWidth = size.contentSize[0]; // tooltip的宽度
            let newX = posX - tooltipWidth / 2; // 设置tooltip的位置，使其在鼠标指针下方
            let parentRect = el.parentNode.getBoundingClientRect();
            let parentWidth = parentRect.width;
            if (newX + tooltipWidth > parentWidth) {
                newX = parentWidth - tooltipWidth;
            }
            let newY = posY + 10; // 增加一定的垂直偏移量
            return [newX, '80%'];
        }
    },
    textStyle: {
        fontSize: 15, // 设置字体大小
    },
    legend: {

    },
    grid: {
        left: '-6%',
        right: '2%',
        bottom: '3%',
        top: '15%',
        containLabel: true
    },
    xAxis: {
        type: 'value',
        // label: {
        //     show: false
        // },
        min:0,
        max: () => xAxisMax,
        show: false,
    },
    yAxis: {
        type: 'category',
        label: {
            show: false
        },
        show: false,
        data: ['攻击类型统计：']
    },
    series: seriesData
}

let optionStd = {
    tooltip: {
        trigger: 'axis',
        axisPointer: {
            // Use axis to trigger tooltip
            type: 'shadow' // 'shadow' as default; can also be 'line' or 'shadow'
        },
        position: function (pos, params, el, elRect, size) {
            let posX = pos[0];
            let posY = pos[1];
            let tooltipWidth = size.contentSize[0]; // tooltip的宽度
            let newX = posX - tooltipWidth / 2; // 设置tooltip的位置，使其在鼠标指针下方
            let parentRect = el.parentNode.getBoundingClientRect();
            let parentWidth = parentRect.width;
            if (newX + tooltipWidth > parentWidth) {
                newX = parentWidth - tooltipWidth;
            }
            let newY = posY + 10; // 增加一定的垂直偏移量
            return [newX, '80%'];
        }
    },
    textStyle: {
        fontSize: 15, // 设置字体大小
    },
    legend: {

    },
    grid: {
        left: '-6%',
        right: '2%',
        bottom: '3%',
        top: '15%',
        containLabel: true
    },
    xAxis: {
        type: 'value',
        // label: {
        //     show: false
        // },
        min:0,
        max: () => MaxStd,
        show: false,
    },
    yAxis: {
        type: 'category',
        label: {
            show: false
        },
        show: false,
        data: ['攻击类型统计：']
    },
    series: seriesDataStd
}

let Xlabel = [];
let Ydata = [];
let time = 0;

function getCurrentTime() {
    var now = new Date();
    var hours = now.getHours();
    var minutes = now.getMinutes();
    var seconds = now.getSeconds();

    // 为了让时、分、秒的显示都是两位数，如果小于10，前面补0
    hours = hours < 10 ? '0' + hours : hours;
    minutes = minutes < 10 ? '0' + minutes : minutes;
    seconds = seconds < 10 ? '0' + seconds : seconds;

    // 将时间格式化为字符串，并返回
    var currentTime = hours + ':' + minutes + ':' + seconds;
    return currentTime;
}

// 更新数据，表格1
function updateData() {
    updateDataPanel(Math.random() * 20 + 50, Math.random() * 1000 + 50)
}

//更新数数据面板的两个图，第一个参数是当前速度，第二个参数是当前数据包的个数
function updateDataPanel(velocityData, packetCountData){
    Ydata.push(velocityData); // 随机生成新数据
    Xlabel.push(getCurrentTime())
    barData.push(packetCountData)
    // 限制数据长度，只保留最近的10个数据点
    if (Xlabel.length > maxItem ) {
        Xlabel.shift(); // 移除最旧的数据点
        Ydata.shift()
        barData.shift();
    }
    myInnerChart2.setOption({ //更新柱状图
        xAxis: {
            data: Xlabel // 更新 x 轴数据
        },
        series: [{
            data: barData // 更新折线图数据
        }]
    });
    setTimeout(()=>{
        myInnerChart.setOption({ // 更新速度图表
            xAxis: {
                data: Xlabel // 更新 x 轴数据
            },
            series: [{
                data: Ydata // 更新折线图数据
            }]
        });
    },200)


}


let barData = []

function updateBarData() {
}


let innerOption = {
    xAxis: {
        min:0,
        max:maxItem-1,
        type: 'category',
        data: Xlabel, // 初始 x 轴数据
        axisLabel: {
            interval: 3, // 强制显示所有刻度标签
        },
        axisTick: {
            //x轴刻度相关设置
            alignWithLabel: true,
        }

    },
    yAxis: {
        name: '当前速率/MB',
        type: 'value',

        splitNumber: 2,
        nameTextStyle: {
            ...AXIS_LABEL_NAME_FONT, // 使用对象扩展合并全局字体样式
            padding: [0, 0, 5, 8] // 其他名称样式属性
        }
    },
    textStyle: AXIS_LABEL_FONT_S,
    grid: {
        left: '12%',
        right: '6%',
        bottom: '10%',
        top:'15%',
    },
    series: [{
        smooth:0.1,
        symbol: "none",
        data: Ydata, // 初始数据
        type: 'line',
        areaStyle: {
            color: {
                type: 'linear',
                x: 0,
                y: 0,
                x2: 0,
                y2: 1,
                colorStops: [{
                    offset: 0, color: 'rgba(58,132,255, 0.35)' // 0% 处的颜色
                }, {
                    offset: 1, color: 'rgba(58,132,255, 0)' // 100% 处的颜色
                }],
                global: false // 缺省为 false
            }
        },
    }]
};

let innerOption2 = {
    xAxis: {
        min:0,
        max:maxItem-1,
        type: 'category',
        data: Xlabel, // 初始 x 轴数据
        axisLabel: {
            interval: 3, // 强制显示所有刻度标签
        },
        axisTick: {
            //x轴刻度相关设置
            alignWithLabel: true,
        }

    },
    yAxis: {
        name: '数据包量/pps',
        type: 'value',
        splitNumber: 2,
        nameTextStyle: {
            ...AXIS_LABEL_NAME_FONT, // 使用对象扩展合并全局字体样式
            padding: [0, 0, 5, 5] // 其他名称样式属性
        }
    },
    textStyle: AXIS_LABEL_FONT_S,
    grid: {
        left: '13%',
        right: '6%',
        bottom: '10%',
        top:'15%',
    },
    series: [{
        data: barData, // 初始数据
        type: 'bar',
        color:'#91cc75',
        // itemStyle: {
        //     borderRadius: [3,3,0,0] // 分别表示左上、右上、右下、左下的圆角半径，可根据需求调整
        // },
        animationEasing: 'quadraticOut',
        barWidth: '65%', // 设置柱子宽度
    }]
};
function deepClone(obj) {
    if (obj === null || typeof obj !== 'object') {
        return obj;
    }

    let clone = Array.isArray(obj) ? [] : {};

    for (let key in obj) {
        if (obj.hasOwnProperty(key)) {
            clone[key] = deepClone(obj[key]);
        }
    }

    return clone;
}


let myChart = null
let myChartStd = null
let myInnerChart = null
let myInnerChart2 = null
let notification


onMounted(() => {
    realTimeDataBak = deepClone(REALTIME_DATA )

    createScroll()
    iniSeriesData()
    // iniSeriesData()
    myChart = echarts.init(chartRef.value, null, {
        height: '140px', // 设置高度
    });
    myInnerChart = echarts.init(innerRef.value, null, {
        height: '250px', // 设置高度
    });
    myInnerChart2 = echarts.init(innerRef2.value, null, {
        height: '250px', // 设置高度
    });

    myInnerChart.setOption(innerOption)
    myInnerChart2.setOption(innerOption2)
    let count = 0
    timer = setInterval(() =>{
        if (count <= 25) {
            readRecord()
            count++
        } else {
            clearInterval(timer)
            setTimeout(()=>{
                notification = ElNotification({
                    title: 'Finish',
                    dangerouslyUseHTMLString: true,
                    message: '<strong>数据集下发完毕！</br>可以和标准数据进行比较</strong>',
                    type: 'success',

                })
                isOver.value = true
            },500)
        }
    }, 1000)
})
</script>

<style scoped>
.title-text{
    color: #595857;
    font-size: 35px;
    font-weight: bold;
    margin-left: 5px;

}
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
.draw-move,
.draw-enter-active, .draw-leave-active {
    transition: all 0.7s cubic-bezier(.25,.09,.36,1.25);
}


.table-container{
    padding: 10px;
    background-color: white;
    border-radius: 12px;
    box-shadow: rgba(0, 0, 0, 0.16) 0px 3px 6px, rgba(0, 0, 0, 0.23) 0px 3px 6px;
}
.panel-container{
    width: 460px;
    margin-left: 20px;
    padding: 10px 0 0;
    background-color: white;
    border-radius: 12px;
    box-shadow: rgba(0, 0, 0, 0.16) 0px 3px 6px, rgba(0, 0, 0, 0.23) 0px 3px 6px;
}

.hint-plug-line{
    background-color: #74b9ff;
    border-radius: 6px;
    color: #F7F7F7;
    /*padding: 0;*/
    padding-left: 4px;
    padding-right: 4px;
    font-weight: bold;
    font-size: 11px;
    line-height: 17px;
    margin-left: 8px;
}
</style>
