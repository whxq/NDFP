<template>
    <el-row >
        <div class="title-container">
            <el-row>
                <div>
                    <div class="first-title-text" style="line-height: 35px; margin-bottom: 5px">
                        联邦学习训练流程
                    </div>
                    <div class="divide-line"> </div>
                </div>

                <div class="hint-plug" v-show="isStart" @click="startAnimation()" style="background-color: #80989b;padding: 4px 7px 4px 6px;">
                    开始模拟训练
                </div>
            </el-row>


        </div>

        <div class="title-container" style="margin-left: auto; align-items: end; margin-right: 210px">
            <div class="first-title-text" style="line-height: 35px; margin-bottom: 5px">
                训练步骤
            </div>
<!--            <div class="divide-line" style="background-color:#74b9ff;"> </div>-->
        </div>
    </el-row>


    <el-container style="height: 100%;">
        <el-main style="padding: 10px; margin-top: 3px">
            <div style="background-color: white; height: 97%; padding: 10px;  box-shadow: rgba(0, 0, 0, 0.16) 0px 3px 6px, rgba(0, 0, 0, 0.23) 0px 3px 6px;border-radius: 15px; z-index: 999">
                <div style="background-color: #2d4059; width: 100%; height: 100%; border-radius: 10px ">
                    <div  style="display: grid; justify-items: center; margin-bottom: 210px; ">
                        <img style="    margin-top: 50px; padding-right: 0px;z-index: 999"
                             :style="genWidth(1070,3940, 0.11)"
                             :src="imageUrls[0]" id="server" />
                    </div>

                    <div style="display: grid; justify-items: center; padding-bottom: 40px;"  >
                        <!--                            <img :style="genWidth(1126, 2240, 0.3)" id="clients" src="../assets/Clients.png"/>-->
                        <div class="container">
                            <div style="z-index: 999">
                                <img id="client1" :style="genWidth(2740,930, 0.125)" :src="imageUrls[1]"/>
                            </div>

                            <div style="z-index: 999">
                                <img id="client2" :style="genWidth(2740,930, 0.125)" :src="imageUrls[2]"/>
                            </div >

                            <div style="z-index: 999">
                                <img id="client3" :style="genWidth(2740,930, 0.125)" :src="imageUrls[3]"/>
                            </div>

                            <div style="z-index: 999">
                                <img id="client4" :style="genWidth(2740,930, 0.125)" :src="imageUrls[4]"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

        </el-main>
        <el-aside width="380">
            <div style="width: 380px;">
                <div style="padding: 25px;">
                    <el-steps direction="vertical" :active="activeNum"
                              finish-status="success"
                    >
                        <el-step title="初始化">
                            <template v-slot:description>
                                <div style="height: 80px; padding-left: 10px">
                                    <div class="step-container"
                                         :style="{ transform: (activeNum == 0)? `scale(1.08)`:`scale(1)`}">
                                        <div style="height: 7px; border-radius: 10px 10px 0 0"
                                             :style="{ backgroundColor: (activeNum === 0)? `#ee6666`:`#95949a`}"></div>
                                        <div class="step-container-inner">
                                            <el-row class="step-card-title" :style="getTitleColor(4)">
                                                <div class="title-deco" style="background-color: #ee6666"></div>
                                                <el-text class="title" :style="{color: (activeNum === 0)? `#595857`:`#95949a`}">
                                                    分发全局模型
                                                </el-text>
                                            </el-row>
                                        </div>

                                    </div>
                                </div>
                            </template>
                        </el-step>


                        <el-step title="开始循环训练">
                            <template v-slot:description>
                                <div style="height: 90px; padding-left: 10px">
                                    <div class="step-container"
                                         :style="{ transform: (activeNum === 1)? `scale(1.08)`:`scale(1)`}">
                                        <div style="height: 7px; border-radius: 10px 10px 0 0"
                                        :style="{ backgroundColor: (activeNum === 1)? `#fc8452`:`#95949a`}"></div>

                                        <div class="step-container-inner">
                                            <el-row class="step-card-title" :style="getTitleColor(4)">
                                                <el-text class="title" :style="{color: (activeNum === 1)? `#595857`:`#95949a`}">
                                                    各客户端训练本地模型
                                                </el-text>
                                            </el-row>
                                        </div>

                                    </div>
                                </div>
                            </template>
                        </el-step>

                        <el-step>
                            <template v-slot:description>
                                <div style="height: 90px; padding-left: 10px">
                                    <div class="step-container"
                                         :style="{ transform: (activeNum == 2)? `scale(1.08)`:`scale(1)`}">
                                        <div style="height: 7px; border-radius: 10px 10px 0 0"
                                             :style="{ backgroundColor: (activeNum === 2)? `#91cc75`:`#95949a`}"></div>

                                        <div class="step-container-inner">
                                            <el-row class="step-card-title" :style="getTitleColor(4)">
                                                <div class="title-deco" style="background-color: #91cc75"></div>
                                                <el-text class="title" :style="{color: (activeNum === 2)? `#595857`:`#95949a`}">
                                                    向服务器发回更新模型
                                                </el-text>
                                            </el-row>
                                        </div>

                                    </div>
                                </div>
                            </template>
                        </el-step>

                        <el-step>
                            <template v-slot:description>
                                <div style="height: 90px; padding-left: 10px">
                                    <div class="step-container"
                                         :style="{ transform: (activeNum == 3)? `scale(1.08)`:`scale(1)`}">
                                        <div style="height: 7px; border-radius: 10px 10px 0 0"
                                             :style="{ backgroundColor: (activeNum === 3)? `#fac858`:`#95949a`}"></div>

                                        <div class="step-container-inner">
                                            <el-row class="step-card-title" :style="getTitleColor(4)">

                                                <el-text class="title" :style="{color: (activeNum === 3)? `#595857`:`#95949a`}">
                                                    中央服务器进行安全聚合
                                                </el-text>
                                            </el-row>
                                        </div>

                                    </div>
                                </div>
                            </template>
                        </el-step>

                        <el-step>
                            <template v-slot:description>
                                <div style="height: 90px; padding-left: 10px">
                                    <div class="step-container"
                                         :style="{ transform: (activeNum == 4)? `scale(1.08)`:`scale(1)`}">
                                        <div style="height: 7px; border-radius: 10px 10px 0 0"
                                             :style="{ backgroundColor: (activeNum === 4)? `#74b9ff`:`#95949a`}"></div>

                                        <div class="step-container-inner">
                                            <el-row class="step-card-title" :style="getTitleColor(4)">
                                                <div class="title-deco" style="background-color: #74b9ff"></div>
                                                <el-text class="title" :style="{color: (activeNum === 4)? `#595857`:`#95949a`}">
                                                    向各客户端发送更新模型
                                                </el-text>
                                            </el-row>
                                        </div>

                                    </div>
                                </div>
                            </template>
                        </el-step>
                    </el-steps>
                </div>
            </div>
        </el-aside>
    </el-container>






</template>

<script setup>
import {onBeforeMount, onBeforeUnmount, onMounted, ref} from "vue";
import VerticalTable from "./VerticalTable.vue";
import {ElNotification, ElTable} from "element-plus";



let activeNum = ref(-1)
let scale = ref(1.1)
let dynamicStyle = ref('')
let i = 0
const items = ref([]);
const deleteItems = ref([1,4,5,7, 10]);
let clientDOMs = []
let tableData = ref([]);
let timerOfTimeLine
let timerOfAnimation
let isStart = ref(true)

const array1 = [];
let showEffectName = 'draw';
// 动画参数
let animOptions = {
    duration: 1000, //持续时长
    timing: 'ease-in', // 动画函数
}
let offset = [-145, -40, 50, 160]
let offsetX = [2, 5, 5, 6]

function genWidth(originalHeight, originalWidth, ratio){
    return 'height:'+originalHeight * ratio+ 'px;width:'+ originalWidth * ratio+'px;'
}

let serverDOM

function iniDOMArray(){
    serverDOM = document.getElementById('server')
    //添加所有的imgDOM对象
    for (i = 1; i <= 4; i++){
        clientDOMs.push({
            index: i,
            dom: document.getElementById('client' + i)
        })
    }
    return clientDOMs
}

let pointPosition = ref([])
function intPointsPosition(){
    const element = document.getElementById('clients').getBoundingClientRect()
    console.log('element',element)
    const left = element.left; // 元素相对于其父元素的左侧位置
    const top = element.top; // 元素相对于其父元素的顶部位置

    console.log('left',left)
    console.log('top',top)

    pointPosition.value[1] = `left: ${left}px; top: 395px;`;
}

function getPointsPosition(num){
    if (pointPosition.value[num] !== undefined) {
        return pointPosition.value[num];
    }
    return ''; // 或者你可以返回一个默认的样式值
}



function genLeaderLine(color, reverse, curIndex, itemOffset = 0) {
    if (clientDOMs.length == 0){
        iniDOMArray()
    }

    let options = {
        dash: { // 虚线样式
            len: 18, // 虚线长度
            gap: 8  // 间隔长度
        },
        color: color,
        startSocket: '' , // 设置起点位置为上方
        endSocket: '',
        endPlugSize: 0.8, // 结束端箭头大小
        path: 'straight',
    }
    if (reverse == false){
        options.startSocket = 'bottom'
        options.endSocket = 'top'
        clientDOMs.forEach((domItem, index) => {
            let serverX = serverDOM.offsetWidth/2 + offset[index] + itemOffset
            let clientX = domItem.dom.offsetWidth/2 + offsetX[index] + itemOffset
            domItem['line' + curIndex] = new LeaderLine(
                LeaderLine.pointAnchor(serverDOM, {
                    x: serverX,
                    y: serverDOM.offsetHeight + 3
                }),
                LeaderLine.pointAnchor(domItem.dom,{
                    x: clientX,
                    y: 0
                }),
                options
            )
            //domItem.line.show(showEffectName, animOptions)
        })
    } else {
        options.startSocket = 'top'
        options.endSocket = 'bottom'
        clientDOMs.forEach((domItem, index) => {
            let serverX = serverDOM.offsetWidth/2 + offset[index] + itemOffset
            let clientX = domItem.dom.offsetWidth/2 + offsetX[index] + itemOffset
            domItem['line' + curIndex] = new LeaderLine(
                LeaderLine.pointAnchor(domItem.dom,{
                    x: clientX,
                    y: 0
                }),
                LeaderLine.pointAnchor(serverDOM, {
                    x: serverX ,
                    y: serverDOM.offsetHeight
                }),
                options
            )
            //domItem.line.show(showEffectName, animOptions)
        })
    }
}

function cleanLeaderLine(){
    clientDOMs.forEach((domItem) => {
        domItem?.line1?.remove()
        domItem?.line2?.remove()
        domItem?.line3?.remove()
    })
}
let preLineIndex = 0

// function setCurActiveLine(lineIndex){
//     setTimeout(() =>{
//         setCurActiveLineRaw(lineIndex)
//     },50)
// }
function setCurActiveLine(lineIndex){
    if (clientDOMs.length != 0 && preLineIndex != 0) {
        clientDOMs.forEach((item) => {
            item['line' + preLineIndex]?.setOptions({ // element-1, element-2
                dash: { // 虚线样式
                    len: 20, // 虚线长度
                    gap: 8,  // 间隔长度
                    animation: false
                }
            })
        })
    }
    clientDOMs.forEach((item) => {
        item['line' + lineIndex]?.setOptions({ // element-1, element-2
            dash: { // 虚线样式
                len: 20, // 虚线长度
                gap: 8,  // 间隔长度
                animation: true,
            },
        })
    })
    preLineIndex = lineIndex
}

let lineArray1 = []
let lineArray2 = []
let lineArray3 = []
let curActive = []
let curAnimation = []
function addAnimations(name, range, duration, pause = 0, times){
    if (name == 'server') {
        addAnimation(name, range, duration, pause, times)
    }else {
        for (let j = 1; j <= 4; j++) {
            console.log('name + i',name + j)
            setTimeout(()=>{
                addAnimation(name + j, range, duration, pause, times)
                },
                duration*1/2*j)
        }
    }
}

function addAnimation(name, range, duration, pause = 0, times = 1){
    const image = document.getElementById(name);
    let scalex = 1.0;
    let scalingUp = true;
    let translateY = 0;
    let maxTranslate = range;

    function singleAnimation() {
        if (scalingUp) {
            translateY -= range; // 向上移动
            if (translateY <= -maxTranslate) {
                scalingUp = false;
            }
        } else {
            translateY += range; // 向下移动
            if (translateY >= 0) {
                scalingUp = true;
            }
        }
        // 应用上下跳跃的变换
        image.style.transition = `transform ${duration / 1000}s cubic-bezier(0.250, 0.15, 0.5, 1.85)`;
        image.style.transform = `translateY(${translateY}px)`;
    }
    let timer
    timer = setInterval(()=>{
        if (times > 1){
            animationStep()
        } else clearInterval(timer)
        times --
    }, 2*duration + pause)
    function animationStep(){
        singleAnimation()
        setTimeout(()=>{
            singleAnimation()
        },duration + 5)
    }
    animationStep()
}

let imageUrls = [
    'src/assets/server.png',
    'src/assets/client1.png',
    'src/assets/client2.png',
    'src/assets/client3.png',
    'src/assets/client4.png',
];

// let imageUrls = ref([])
const preloadImages = (urls) => {
    urls.forEach(url => {
        const img = new Image();
        img.src = url;
        imageUrls.value.push(url);
    });
};

function startAnimation(){
    let animationDuration = 200
    let animationPause = 650
    let executeTimes = 2
    let period = 2*animationDuration + animationPause
    let flag = 0
    isStart.value = false



    setCurActiveLine(1)
    let round = 0
    activeNum.value = 0
    timerOfTimeLine = setInterval(()=> {
        flag %= 4
        if (flag == 0) {

            setCurActiveLine(0) //0用于停止连线
            setTimeout(()=>{
                addAnimations('client',-18, animationDuration, animationPause, executeTimes)

            },100)

            activeNum.value = 1
        } else if (flag == 1){
            setTimeout(()=>{
                setCurActiveLine(2)
            },100)

            activeNum.value = 2
        } else if (flag == 2){
            setCurActiveLine(0)
            setTimeout(()=>{
                addAnimations('server',10, animationDuration, animationPause, executeTimes)

            },100)

            activeNum.value = 3
        } else if (flag == 3){
            setTimeout(()=>{
                setCurActiveLine(3)
            },100)
            activeNum.value = 4
            setTimeout(()=>{
                round++
                if (round > 0){
                    notification = ElNotification({
                        title: 'Finish',
                        dangerouslyUseHTMLString: true,
                        message: '<strong>第'+round +'轮学习完成，即将进行第'+(round+1) +'轮学习</strong>',
                        type: 'success',
                        // duration:0
                    })
                }

            }, executeTimes*period + 100)
        }
        flag ++
    }, executeTimes*period + 300)
}
let notification

onMounted(()=>{
    genLeaderLine( '#ee6666', false,1, -20)
    genLeaderLine('#91cc75', true, 2)
    genLeaderLine('#74b9ff', false,3, 20)


})

onBeforeUnmount(()=>{
    cleanLeaderLine()
    clearInterval(timerOfTimeLine)
    clearInterval(timerOfAnimation)
    notification?.close()
})

function getTitleColor(num) {
    // if (activeNum.value != num){
    //     // return { backgroundColor: 'rgba(255, 45, 81)' }
    //     return { backgroundColor: '#3498db' }
    // }
}



</script>

<style scoped>
.step-card-title{
    font-size: 20px;
    border-radius: 10px 10px 0 0;
    padding-bottom: 0;
    padding-left: 0;
    font-weight: bolder;

}

.step-container{
    transition: transform 0.4s cubic-bezier(.25,.09,.36,1.25);
    background-color: #f9f9f9;
    border-radius: 10px;
    box-shadow: rgba(0, 0, 0, 0.16) 0px 3px 6px, rgba(0, 0, 0, 0.23) 0px 3px 6px;
    width: 285px;
    height: min-content;
}

.step-container-inner{
    padding: 8px;
    padding-top: 6px;
}

.title-deco{
    height: 9px;
    width: 26px;
    border-radius: 4px;
    margin-top: 8px;
    margin-right: 2px;
}

content{
    font-size: 15px;
}


.title{
    /*background-color: #96ce54;*/
    color: #595857;
    font-size: 21px;
    border-radius: 5px;
    padding: 2px 5px;
}

.el-step__main{
    height: min-content!important;
}

:deep(.step-card){
    font-size: 20px;
}

:deep(.el-step__main){
    height: min-content!important;
}

:deep(.el-step__title is-success){
    font-weight: bold!important;
}

.vertical-table{
    margin-bottom: 5px;
}


.container {
    display: grid;
    grid-template-columns: repeat(4, 1fr); /* 创建四列，每列宽度相等 */
    gap: 65px; /* 元素之间的间距 */
}


</style>

<style>
.el-step__title.is-success {
    font-weight: bold;

}
</style>