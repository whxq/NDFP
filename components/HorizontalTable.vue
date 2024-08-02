<template>
    <table class="horizontal-table">
        <tbody>
        <tr>
            <td v-for="(item, index) in items.slice(0, maxLength)" :key="index" nowrap class="label" :class="{ 'highlighted': deleteItems?.includes(index) }">
                <div style="width: 75px">
                    {{ '特征值' }}{{ index + 1 }}
                </div>
            </td>
        </tr>
        <tr>
            <td v-for="(item, index) in items.slice(0, maxLength)" :key="index" nowrap class="value" :class="{ 'highlighted': deleteItems?.includes(index) }">
<!--                v-if="!deleteItems?.includes(index)"-->
                <div  style="width: 75px;">
                    {{ item }}
                </div>
            </td>
        </tr>
        </tbody>
    </table>
</template>

<script setup>
import { ref } from 'vue';
const maxLength = 20
let props = defineProps(['items','deleteItems'])

function moreItemsAvailable() {
    return props.items.length > 2*maxLength;
}

function packArrayItems() {
    const packedArray = [];

    for (let i = 0; i < props.items.length; i += 2) {
        // 创建一个包含两个元素的对象
        const obj = {};
        obj.elm1 = props.items[i];

        // 如果存在下一个元素，则将其添加到对象中
        if (i + 1 < props.items.length) {
            obj.elm2 = props.items[i + 1];
        }

        // 将打包后的对象压入新数组
        packedArray.push(obj);
    }

    return packedArray;
}


</script>

<style scoped>
.horizontal-table {
    background-color: white;
    border-collapse: collapse;
    width: 100%;
}

.horizontal-table td {
    border-top: 1px solid #e4e7ed; /* 设置上边框 */
    border: 1.2px solid #e4e7ed; /* 设置下边框 */
    /*border-left: none; !* 删除左边框 *!*/
    /*border-right: none; !* 删除右边框 *!*/
    padding: 8px;
    text-align: center;
}

.horizontal-table .label {
    font-weight: bold;
    text-align: center;
    color: gray;
    font-size: 15px;
}
.horizontal-table .value {
    text-align: center;
    color: #595857;
    font-size: 15px;
}

.highlighted {
    background-color:  #74b9ff; /* 将背景颜色设置为红色 */

}

.highlighted div {
    color: #F0F0F0;/* 将子元素文本颜色设置为白色 */

    text-decoration: line-through;
    text-decoration-thickness: 2px;
    text-decoration-color: #74b9ff;

}
</style>
