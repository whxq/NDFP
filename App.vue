<template>
    <el-container>
        <el-aside  class="el-aside-container">
            <Menu :change-cur-component-id="changeCurComponentId"></Menu>
        </el-aside>
        <el-main>
    <!--            <el-col class="el-main" >-->
    <!--                -->
    <!--            </el-col>-->
            <div class="main-container">
                <home-page v-if="curComponentId == 0"></home-page>
                <OriginalData v-if="curComponentId == 1"/>

                <!--                <TrainingStep v-if="curComponentId == 2"/>-->
                <DichotomyModel v-if="curComponentId === 31"/>
                <AttackStreamStatistics v-if="curComponentId === 32"/>
                <AccuracyRate v-if="curComponentId == 4"/>
                <AttackStatistics v-if="curComponentId == 5"/>
                <training-step2 v-if="curComponentId == 2 "/>
                <RealTimeStreamDisplay v-if="curComponentId == 42"></RealTimeStreamDisplay>
            </div>
        </el-main>


    </el-container>




</template>


<script setup>
import * as echarts from 'echarts';
import {onBeforeMount, onMounted, ref} from 'vue';
import Menu from "./components/Menu.vue";
import AccuracyRate from "./components/AccuracyRate.vue";

import AttackStatistics from "./components/AttackStatistics.vue";
import OriginalData from "./components/OriginalData.vue";
import HomePage from "./components/HomePage.vue";
import DichotomyModel from "./components/DichotomyModel.vue";

import AttackStreamStatistics from "./components/AttackStreamStatistics.vue";
import TrainingStep2 from "./components/TrainingStep.vue";
import RealTimeStreamDisplay from "./components/RealTimeStreamDisplay.vue";

const chartRef = ref(null);
let curComponentId = ref(0)

function changeCurComponentId(id){
    curComponentId.value = id

}

let imageUrls = [
    'src/assets/server.png',
    'src/assets/client1.png',
    'src/assets/client2.png',
    'src/assets/client3.png',
    'src/assets/client4.png',
];

let preloadedImages = ref([])
const preloadImages = (urls) => {
    urls.forEach(url => {
        const img = new Image();
        img.src = url;
        preloadedImages.value.push(img);
    });
};

//图片预加载
onMounted(() => {
    preloadImages(imageUrls)
    console.log('666', typeof preloadedImages.value[0])
})

</script>



<style scoped>
.el-aside-container{
    margin: 10px;
    margin-left: 15px;
    margin-right: 8px;
    border-radius: 20px;
    box-shadow: rgb(38, 57, 77) 0px 10px 20px -10px;
    width: 245px;
    height: min-content;
    padding-right: 0!important;
}

.aside-container{
    padding-right: 0!important;
}
.el-main{
    margin: 10px;
    border-radius: 20px;
    margin-right: 15px;
    background-color: rgba(247, 247, 247, 0.92);
    box-shadow: rgb(38, 57, 77) 0px 10px 20px -10px;
    height: 895px;
}
.main-container{
    height: 95% !important;
}


</style>
