<template>
    <div class="title-container">
        <div class="first-title-text" style="font-size: 40px">基于联邦学习和P4的智能高效网络入侵检测系统</div>
        <div class="sub-title-text">
<!--            An intelligent and efficient Network intrusion Detection system based on <br>Federated learning and P4-->
            An Intelligent and Efficient NIDS based on Federated Learning and P4
        </div>
        <div class="divide-line" > </div>
    </div>


    <el-row gutter="20" style="height: 88%">
        <el-col id="time-line-deco" span="12" style="margin-right: 35px">
            <el-image style="width: 540px; position: absolute; transform: translateX(40px) translateY(-60px)" src="src/assets/home_page3.svg" fit="fill"/>
            <el-image style="width: 380px; position: absolute; transform: translateX(208px) translateY(160px)" src="src/assets/home_page.svg" fit="fill"/>
        </el-col>

        <el-col  span="8" id="time-line-deco">

            <div style="width: max-content; align-items: end; margin-left: 600px; margin-top: 70px" class="info-container">



                <div style="padding-right: 20px; padding-left: 20px;">

                    <el-row style="margin-bottom: 40px; margin-top: 25px">
                        <!--                    <el-image style="width: 40px" src="src/assets/icon1.svg" fit="fill"/>-->
                        <div class="title-deco" style="background-color: #5470c6;"></div>
                        <el-text class="title-text">
                            基于EA算法与深度自编码器的数据泛化
                        </el-text>
                    </el-row>

                    <el-row style="margin-bottom: 40px">
                        <!--                    <el-image style="width: 40px" src="src/assets/icon2.svg" fit="fill"/>-->
                        <div class="title-deco" style="background-color: #91cc75;"></div>
                        <el-text class="title-text">
                            结合深度自编码器与联邦学习的分布式训练
                        </el-text>
                    </el-row>

                    <el-row style="margin-bottom: 40px">
                        <!--                    <el-image style="width: 40px" src="src/assets/icon3.svg" fit="fill"/>-->
                        <div class="title-deco" style="background-color: #fac858"></div>
                        <el-text class="title-text">
                            基于逆Potts的图型结构存储异常流量信息
                        </el-text>
                    </el-row>

                    <el-row style="margin-bottom: 40px">
                        <!--                    <el-image style="width: 40px" src="src/assets/icon4.svg" fit="fill"/>-->
                        <div class="title-deco" style="background-color: #ee6666;"></div>
                        <el-text class="title-text">
                            基于EFC的多分类算法检测网络攻击类型
                        </el-text>
                    </el-row>


                    <el-row style="margin-bottom: 25px">
                        <!--                    <el-image style="width: 40px" src="src/assets/icon5.svg" fit="fill"/>-->
                        <div class="title-deco" style="background-color: #73c0de;"></div>
                        <el-text class="title-text">
                            基于可编程数据平面的高效数据包检测
                        </el-text>
                    </el-row>

                </div>


            </div>
        </el-col>



    </el-row>



</template>

<script setup>

</script>

<style scoped>
.title-container{
    width: max-content;
}
.title-deco{
    background-color: #96ce54;
    height: 30px;
    width: 7px;
    border-radius: 5px;
    margin-right: 8px;
    padding-top: 4px;
    transform: translateY(2px);
}



#time-line-deco{
    width: min-content;
    height: 85%;
    display: flex;
    align-items: center; /* 垂直居中 */
}

.title-text{

    color: #595857;
    font-size: 26px;
    font-weight: bold;
}

.info-container{

    background-color: rgba(247, 247, 247, 0.8);
    border-radius: 20px;
    /*box-shadow: rgba(0, 0, 0, 0.1) 0px 4px 6px -1px, rgba(0, 0, 0, 0.06) 0px 2px 4px -1px;*/
    box-shadow: rgb(38, 57, 77) 0px 10px 20px -10px;
    height: min-content;
}

.sub-title-text{
    color: #595857;
    font-size: 24.3px;
    font-weight: bold;
    padding-right: 5px;
    padding-left: 5px;
}
</style>