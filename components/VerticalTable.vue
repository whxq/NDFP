<template>
    <table class="vertical-table">
        <tbody>
        <tr v-for="(item, index) in packArrayItems().slice(0, maxLength)  " :key="index" >
            <td nowrap class="label" :class="{ 'highlighted': deleteItems?.includes(2*index + 1) }">
                <div style="width: 60px">
                    {{' 特征值'}}{{2*index + 1}}
                </div>
            </td>
            <td nowrap class="value" :class="{ 'highlighted': deleteItems?.includes(2*index + 1) }">
                <div style="width: 50px;">
                    {{item?.elm1 }}
                </div>
            </td>
            <td nowrap class="label" :class="{ 'highlighted': deleteItems?.includes(2*index + 2) }">
                <div style="width: 60px">
                    {{' 特征值'}}{{2*index + 2}}
                </div>
            </td>
            <td nowrap class="value" :class="{ 'highlighted': deleteItems?.includes(2*index + 2) }">
                <div style="width: 50px">
                    {{item?.elm2 }}
                </div>
            </td>
        </tr>

<!--        <tr v-if="moreItemsAvailable()">-->
<!--            <td colspan="2" class="label">......</td>-->
<!--        </tr>-->
        </tbody>
    </table>
</template>

<script setup>
import { ref } from 'vue';
const maxLength = 6
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
.vertical-table {
    background-color: white;
    border-collapse: collapse;
    width: 100%;
}

.vertical-table td {
    border-top: 1px solid #e4e7ed; /* 设置上边框 */
    border-bottom: 1px solid #e4e7ed; /* 设置下边框 */
    border-left: none; /* 删除左边框 */
    border-right: none; /* 删除右边框 */
    padding: 8px;
    text-align: center;
}

.vertical-table .label {
    font-weight: bold;
    text-align: center;
    color: #595857;
}
.vertical-table .value {
    text-align: center;
    color: #595857;
}
.highlighted {
    background-color:  #a1afc9; /* 将背景颜色设置为红色 */

}
.highlighted div {
    /*color: #F7F7F7;!* 将子元素文本颜色设置为白色 *!*/
    text-decoration: line-through;
    text-decoration-thickness: 2px;
    text-decoration-color: #595857;

}
</style>
