<template>
    <select size="5">
        <option v-for="propertyValue in propertyValueList" :key="propertyValue">{{ propertyValue }}</option>
    </select>
    <label>属性: <input v-model="propertyInput"></label>
    <label>值: <input v-model="valueInput"></label>
    <button @click="createProperty">创建</button>
    <button @click="uploadProperty">上传</button>
</template>
<script setup lang="ts">
import { ref,reactive } from "vue"

const propertyValueList:string[] = reactive([])
let valueList:string[] = []
let propertySet = new Set<string>()
let propertyMap = new Map()
const propertyInput = ref("")
const valueInput = ref("")

// function SetPropertySet(propertySet: Set<string>, property: string[]): Set<string> {
//     for (let i = 0; i < property.length; i++) {
//         propertySet.add(property[i])
//     }
//     return propertySet
// }

function SetPropertyMap(propertyMap: Map<string, string>, propertySet: Set<string>, value: string[]): Map<string, string> {
    const propertyArray = [...propertySet]
    for (let i = 0; i < propertySet.size; i++) {
        const _property = propertyArray[i]
        const _value = value[i]
        propertyMap.set(_property, _value)
    }
    return propertyMap
}

function createProperty() {
    if(propertyInput.value.trim() && valueInput.value.trim()) {
        const pair:string = `${propertyInput.value}：${valueInput.value}`
        if(!propertySet.has(propertyInput.value)) {
            propertySet.add(propertyInput.value)
            valueList.push(valueInput.value)
            propertyValueList.push(pair)
            propertyInput.value = ""
            valueInput.value = ""
        }
    }
}

function uploadProperty() {
    SetPropertyMap(propertyMap,propertySet,valueList)
    propertyValueList.splice(0)
    valueList.splice(0)
    propertySet.clear()
    alert("上传成功")
    console.log(propertyMap);
}


</script>
<style>
input {
  display: block;
  margin-bottom: 10px;
}

select {
  float: left;
  margin: 0 1em 1em 0;
  width: 14em;
}

.buttons {
  clear: both;
}
</style>