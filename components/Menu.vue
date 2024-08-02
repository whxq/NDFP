<template>
    <el-col class="aside-container">

        <el-row style="height: 55px" @click="changeCurComponentId(0)">
            <el-text class="title-text">NDFP</el-text>
            <el-image style="height: 55px; margin-left: 12px" src="src/assets/img.png" fit="fill" id="time-line-deco"/>
<!--            <el-image style="height: 55px; margin-left: 18px" src="src/assets/menu_icon.png" fit="fill" id="time-line-deco"/>-->
        </el-row>
        <div class="divide-line"/>
        <el-col>

            <el-menu
                    style="background-color: transparent"
                    class="el-menu-vertical-demo"
                    @open="handleOpen"
                    @close="handleClose"
            >


                <el-menu-item index="1" style="padding: 10px" @click="changeCurComponentId(1)" class="row-text">
                    数据集展示
                </el-menu-item>

                <el-menu-item index="2" style="padding: 10px" @click="changeCurComponentId(2)" class="row-text">
                    模型训练流程
                </el-menu-item>

                <el-sub-menu index="3" style="background-color: transparent!important; padding-right: 0px ">
                    <template #title >
                        <div class="row-text" style="background-color: transparent!important;">异常流量检测</div>
                    </template>
                    <el-menu-item index="3-1" class="row-text" style="padding: 25px"  @click="changeCurComponentId(31)" >阈值收敛过程</el-menu-item>
                    <el-menu-item index="3-2" class="row-text" style="padding: 25px" @click="changeCurComponentId(32)" >检测效果评估</el-menu-item>
                </el-sub-menu>

<!--                <el-menu-item index="6" style="padding: 5px" @click="changeCurComponentId(5)" class="row-text">-->
<!--                    攻击类型检测模块-->
<!--                </el-menu-item>-->

                <el-sub-menu index="4" style="background-color: transparent!important; padding-right: 0px ">
                    <template #title >
                        <div class="row-text" style="background-color: transparent!important;">攻击类型检测</div>
                    </template>
                    <el-menu-item index="4-1" class="row-text" style="padding: 25px"  @click="changeCurComponentId(5)" >检测效果统计</el-menu-item>
                    <el-menu-item index="4-2" class="row-text" style="padding: 25px" @click="changeCurComponentId(42)" >实时数据流统计</el-menu-item>
                </el-sub-menu>

            </el-menu>
        </el-col>
    </el-col>
</template>

<script setup>
import {ref} from 'vue'

defineProps(["changeCurComponentId"])


</script>

<style scoped>

.aside-container{
    padding-left: 5%;
    padding-bottom: 2%;
    padding-top: 2%;
    /*padding-right: 0;*/
    background-color: rgba(247,247,247,0.85);
    border-radius: 15px;
}
.title-text{
    color: #595857;
    font-size: 39px;
    font-weight: bold;
    margin-right: 22px;
    margin-top: 11px;
    height: min-content;
}

.el-menu{
    background-color: rgba(247,247,247,0.85);
}

.row-text{
    color: #595857;
    font-size: 25px;
    font-weight: bold;
    background-color: transparent!important;
}

:deep(.el-menu el-menu--inline){
    background-color: transparent!important;
}
:deep(.el-sub-menu__title:hover){
    background-color: rgba(236, 239, 241, 0.5) !important;
}

    :deep(.el-sub-menu__title){
        padding: 10px !important;
        padding-right: 0!important;
    }
:deep(.el-icon svg){
    position: absolute;
}

:deep(ul){
    background-color: transparent!important;
}

:deep(.el-menu-item:hover){
    background-color: rgba(236, 239, 241, 0.5)!important;
}

.divide-line{
    background-color: #595857;
    height: 3px;
    width: 95%;
    border-radius: 4px;
    /*margin-bottom: 5px;*/
    margin-top: 2px;

}
/deep/ .el-menu {
    border-right: 0;
    width: 230px;
}
:deep(.el-icon){
    margin-left: 50px;
}
#time-line-deco{
    display: flex;
}

/deep/.el-menu-item{
    background: transparent!important;
}


</style>