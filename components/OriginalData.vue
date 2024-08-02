<template>
    <el-row>
        <div class="title-container">

            <el-dropdown placement="bottom-start" class="custom-dropdown">

                <div class="first-title-text" style="margin-bottom: 5px">
                    训练数据集: {{curTitle + ' Samples'}}
                </div>
                <template #dropdown>
                    <el-dropdown-menu class="custom-dropdown-menu">
                        <el-dropdown-item :class="[activeNum == 1 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('CIC-IDS-2018',1)">CIC-IDS-2018 Samples</el-dropdown-item>
                        <el-dropdown-item :class="[activeNum == 2 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('UNSW-NB15',2)">UNSW-NB15 Samples</el-dropdown-item>
                        <el-dropdown-item :class="[activeNum == 3 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('Bot-IoT',3)">Bot-IoT Samples</el-dropdown-item>
                        <el-dropdown-item :class="[activeNum == 4 ? 'choice-item-selected' : 'choice-item']" @click="setTitle('ToN-IoT',4)">ToN-IoT Samples</el-dropdown-item>
                    </el-dropdown-menu>
                </template>
            </el-dropdown>
            <div class="divide-line"> </div>
        </div>
        <div class="hint-plug" @click="changeToNext()" style="background-color: #80989b;padding: 4px 7px 4px 6px; align-items: end; margin-left: auto; margin-right: 0px">
            下一个数据集
        </div>
    </el-row>

    <div ref="chartRef" style="width: 100%;margin-top: 1%; height: 125px;"></div>
    <el-scrollbar style="height: 83%; margin-top: 0%">
        <div>
            <el-table :data="originalData" border stripe ref="myTable"
                      >
                <el-table-column
                    v-for="(column, columnIndex) in originalData[0]"
                    :key="columnIndex"
                    align="center"
                    :width="tableTitleWidth[columnIndex]"
                    :label="tableTitle[columnIndex]">
                    <template v-slot="{ row }">
                        <div>{{ row[columnIndex] }}</div>
                    </template>
                </el-table-column>
            </el-table>
        </div>
    </el-scrollbar>

</template>

<script setup>
import {onBeforeMount, onMounted, ref, watch} from 'vue';
import { ElTable } from 'element-plus';
import * as echarts from "echarts";
import {ORIGINAL_DATA, TRAINING_DATA, TRAINING_DATA_TYPE} from "../const/data.js";
const chartRef = ref(null);
let tableData = ref([]);
let count = 0
let curTitle = ref('Bot-IoT')
let activeNum = ref(-1)
let trainingDataType = TRAINING_DATA_TYPE
    let totalTitle = ['','CIC-IDS-2018','UNSW-NB15','Bot-IoT','ToN-IoT']

let tableTitle = [
    'IPV4_SRC_ADDR', 'L4_SRC_PORT', 'IPV4_DST_ADDR', 'L4_DST_PORT', 'PROTOCOL',
    'L7_PROTO', 'IN_BYTES', 'IN_PKTS', 'OUT_BYTES', 'OUT_PKTS', 'TCP_FLAGS',
    'CLIENT_TCP_FLAGS', 'SERVER_TCP_FLAGS', 'FLOW_DURATION_MILLISECONDS', 'DURATION_IN',
    'DURATION_OUT', 'MIN_TTL', 'MAX_TTL', 'LONGEST_FLOW_PKT', 'SHORTEST_FLOW_PKT',
    'MIN_IP_PKT_LEN', 'MAX_IP_PKT_LEN', 'SRC_TO_DST_SECOND_BYTES', 'DST_TO_SRC_SECOND_BYTES',
    'RETRANSMITTED_IN_BYTES', 'RETRANSMITTED_IN_PKTS', 'RETRANSMITTED_OUT_BYTES', 'RETRANSMITTED_OUT_PKTS',
    'SRC_TO_DST_AVG_THROUGHPUT', 'DST_TO_SRC_AVG_THROUGHPUT', 'NUM_PKTS_UP_TO_128_BYTES',
    'NUM_PKTS_128_TO_256_BYTES', 'NUM_PKTS_256_TO_512_BYTES', 'NUM_PKTS_512_TO_1024_BYTES',
    'NUM_PKTS_1024_TO_1514_BYTES', 'TCP_WIN_MAX_IN', 'TCP_WIN_MAX_OUT', 'ICMP_TYPE',
    'ICMP_IPV4_TYPE', 'DNS_QUERY_ID', 'DNS_QUERY_TYPE', 'DNS_TTL_ANSWER',
    'FTP_COMMAND_RET_CODE', 'Label', 'Attack', 'Dataset', 'efc'
]
let tableTitleWidth
let originalData = ref([])
let myTable = ref(null)
let tableKeys = []
let tableLabels = []

function calculateColumnWidths(titles) {
    const charWidth = 10; // 每个字符的平均宽度（根据实际情况调整）
    const padding = 25; // 额外的内边距宽度
    return titles.map(title => {
        if (title == 'Attack') {
            return 100
        } else if (title == 'Dataset') {
            return 200
        } else if (title == 'efc') {
            return 80
        }

        return title.length * charWidth + padding}
    );
}
tableTitleWidth = calculateColumnWidths(tableTitle)

defineProps(['preloadedImages'])

function changeToNext(){
    generateRandomArray()
    if (activeNum.value < 4){
        activeNum.value++
    } else {
        activeNum.value = 1
    }
    setTitle(totalTitle[activeNum.value], activeNum.value)
}

function setTitle(newTitle, newActiveNum){
    Object.keys(trainingDataType).forEach(function(key) {
        if (key === newTitle) {
            curData = TRAINING_DATA_TYPE[key]
        }
    })
    ORIGINAL_DATA.forEach((item) => {
        if (item.name == newTitle) {
            originalData.value = item.data
            // if (originalData.length > 0) {
            //     tableKeys = Object.keys(jsonData[0]);
            //     tableLabels = this.generateLabels(this.tableKeys);
            // }
        }

    })

    // //创建滚动对象
    // clearScroll()
    // const table = myTable.value.layout.table.refs;
    // const tableWrapper = table.bodyWrapper.firstElementChild.firstElementChild
    // tableWrapper.scrollLeft = 0
    // setTimeout(()=>{
    //     createScroll()
    // }, 1200)


    activeNum.value = newActiveNum
    curTitle.value = newTitle
}

let curData = null
let seriesData = [];
let xAxisMax = 0

function iniSeriesData() {
    Object.keys(curData).forEach(function(key) {
        let seriesItem = {
            name: key,
            type: 'bar',
            stack: 'total',
            label: {
                show: false
            },
            emphasis: {
                focus: false,  // 关闭鼠标悬停时的高亮效果
                blur: false  // 关闭其他柱子颜色消失的效果
            },
            data: [curData[key]]
        };
        xAxisMax += curData[key]
        seriesData.push(seriesItem);
    })
}

function cleanAttributes(){
    xAxisMax = 0
    seriesData.length = 0
}

watch(()=>activeNum, ()=>{
    cleanAttributes()
    iniSeriesData()

    console.log('xAxisMax',xAxisMax)
    myChart.setOption(option , true);
}, {deep: true})


let option = {
    tooltip: {
        trigger: 'axis',
        axisPointer: {
            // Use axis to trigger tooltip
            type: 'shadow' // 'shadow' as default; can also be 'line' or 'shadow'
        },
        position: function (pos, params, el, elRect, size) {
            var posX = pos[0];
            var posY = pos[1];
            var tooltipWidth = size.contentSize[0]; // tooltip的宽度

            var newX = posX - tooltipWidth / 2; // 设置tooltip的位置，使其在鼠标指针下方
            var parentRect = el.parentNode.getBoundingClientRect();
            var parentWidth = parentRect.width;
            if (newX + tooltipWidth > parentWidth) {
                newX = parentWidth - tooltipWidth;
            }
            var newY = posY + 10; // 增加一定的垂直偏移量

            return [newX, '80%'];
        }

    },
    textStyle: {
        fontSize: 15, // 设置字体大小
    },
    legend: {},
    grid: {
        left: '-6.6%',
        right: '0%',
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
        data: ['数据包类型：']
    },
    series: seriesData
};
function generateRandomArray() {
    tableData.value.length = 0
    for (let i = 0; i < 20; i++) {
        const row = [];
        for (let j = 0; j < 20; j++) {
            row.push(((Math.random() * 100)/100).toFixed(4));
        }
        tableData.value.push(row);
    }
}

let scrollTimer = null
const clearScroll = () => {
    clearInterval(scrollTimer)
    scrollTimer = null
}

const createScroll = () => {
    clearScroll();
    // 拿到 table
    const table = myTable.value.layout.table.refs;
    // 拿到可以滚动的元素
    const tableWrapper = table.bodyWrapper.firstElementChild.firstElementChild;

    scrollTimer = setInterval(() => {

        // 判断是否滚动到右边，如果到右边了置为0（可视宽度+距离左边=整个宽度）
        if (tableWrapper.clientWidth + tableWrapper.scrollLeft >= tableWrapper.scrollWidth) {
            tableWrapper.scrollLeft = 0; // 滚动到右边后重置到最左边
        } else {
            tableWrapper.scrollLeft += 1; // 否则继续向右滚动
        }
    }, 5);
}


let myChart = null


onMounted(() => {
    myChart = echarts.init(chartRef.value, null, {
        height: '145px', // 设置高度
    });
    setTitle('CIC-IDS-2018',1 )
})
onBeforeMount(() => {
    clearScroll()
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
.el-scrollbar__bar.is-vertical {
    opacity: 1;
}
</style>

<style>
.el-scrollbar__bar.is-vertical {
    opacity: 1;
}
</style>
