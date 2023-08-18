<script setup lang="ts">
import { reactive, ref } from 'vue'

const startDate: number = 9000000000

const hourList: number[] = Array.from(new Array(24).keys())
const minuteList: number[] = Array.from(new Array(60).keys())
const secondList: number[] = Array.from(new Array(60).keys())

//日期与时分秒转Date类型
function stringToDate(str: any, hour: number, minute: number, second: number): Date {
    const [y, m, d] = str.split('-')
    return new Date(+y, m - 1, +d, hour, minute, second, 0)
}

//将公元纪年转换为以-9000000000年为基准的绝对日期，输出为Map<number,number>类型
function transformDate(dateInput: Date): Map<number, number> {
    let dateOutput: Map<number, number> = new Map()
    let second: number = dateInput.getSeconds()
    let minute: number = dateInput.getMinutes()
    let hour: number = dateInput.getHours()
    let date: number = dateInput.getDate()
    let month: number = dateInput.getMonth() + 1
    let year: number = dateInput.getFullYear() + startDate
    let tempYear: number[] = []
    for (let i = 0; i < 10; i++) {
        let value: number = year % 10
        tempYear.push(value)
        year = Math.trunc(year / 10)
    }
    tempYear.reverse()
    for (let i = 0; i < 10; i++) {
        dateOutput.set(i, tempYear[i])
    }
    dateOutput.set(10, month)
    dateOutput.set(11, date)
    dateOutput.set(12, hour)
    dateOutput.set(13, minute)
    dateOutput.set(14, second)

    return dateOutput
}

//Map<number,number>转对象类型
function mapToObj(map: Map<number, number>) {
    let obj = Object.create(null);
    for (let [k, v] of map) {
        obj[k] = v;
    }
    return obj;
}

//根据经纬度得到geohash值，输出为字符串
function geoHash(lng: number, lat: number): string {
    let lngList: number[] = []
    let latList: number[] = []
    let totalList: number[] = []
    let left: number = -180.0, right: number = 180.0, mid: number
    let iter: number = 0
    let maxIteration: number = 15
    const Base32 = function (str: any) {
        let num: any = str
        if (typeof num == "string") {
            num = Number(str)
        }
        if (num >= 0 && num <= 9) {
            return num.toString()
        } else if (num >= 10 && num <= 16) {
            return String.fromCharCode(98 + str - 10)
        } else if (num >= 17 && num <= 18) {
            return String.fromCharCode(106 + str - 17)
        } else if (num >= 19 && num <= 20) {
            return String.fromCharCode(109 + str - 19)
        } else if (num >= 21 && num <= 31) {
            return String.fromCharCode(112 + str - 21)
        }
    }

    while (iter < maxIteration) {
        mid = (left + right) / 2.0
        if (lng > mid) {
            lngList.push(1)
            left = mid
        }
        else {
            lngList.push(0)
            right = mid
        }
        iter++
    }
    iter = 0, left = -90.0, right = 90.0
    while (iter < maxIteration) {
        mid = (left + right) / 2.0
        if (lat > mid) {
            latList.push(1)
            left = mid
        }
        else {
            latList.push(0)
            right = mid
        }
        iter++
    }
    iter = 0
    while (iter < maxIteration) {
        totalList.push(lngList[iter])
        totalList.push(latList[iter])
        iter++
    }
    let geohash = ''
    for (let k = 0; k < totalList.length; k = k + 5) {
        let arr = totalList.slice(k, k + 5)
        let sum = parseInt(arr.join(''), 2)
        let base32 = Base32(sum)
        geohash += base32
    }
    return geohash
}
interface timeData {
    date: Date,
    hour: number,
    minute: number,
    second: number
}
interface spatialData {
    lng: number,
    lat: number
}

const newTimeData = reactive<timeData>(
    {
        date: new Date(),
        hour: 0,
        minute: 0,
        second: 0
    }
)

const newSpatialData = reactive<spatialData>(
    {
        lng: 0,
        lat: 0
    }
)

function uploadData() {
    if (newSpatialData.lng > 180.0 || newSpatialData.lng < -180.0 || newSpatialData.lat > 90.0 || newSpatialData.lat < -90.0)
        alert("经度或维度错误！")
    else {
        const date: Map<number, number> = transformDate(stringToDate(newTimeData.date, newTimeData.hour, newTimeData.minute, newTimeData.second))
        const dateObj = mapToObj(date)
        const geohash: string = geoHash(newSpatialData.lng, newSpatialData.lat)
        const data = { "time": dateObj, "spatial": geohash }
        const jsonData: string = JSON.stringify(data)
        newTimeData.date = new Date()
        newTimeData.hour = 0
        newTimeData.minute = 0
        newTimeData.second = 0
        newSpatialData.lng = 0
        newSpatialData.lat = 0
        alert("提交成功")
        console.log({ jsonData })
    }
}

</script>

<template>
    <h1>注册器</h1>
    <div id="registration">
        <h3>时间:</h3>
        <h3>
            <input type="date" v-model="newTimeData.date">
            <select v-model="newTimeData.hour">
                <option v-for="item in hourList">{{ item }}</option>
            </select>
            时
            <select v-model="newTimeData.minute">
                <option v-for="item in minuteList">{{ item }}</option>
            </select>
            分
            <select v-model="newTimeData.second">
                <option v-for="item in secondList">{{ item }}</option>
            </select>
            秒
        </h3>
    </div>
    <h3>地点:</h3>
    <h3>
        经度：<input type="number" placeholder="经度" v-model="newSpatialData.lng">
        纬度：<input type="number" placeholder="纬度" v-model="newSpatialData.lat">
    </h3>
    <button @click="uploadData">提交</button>
</template>

<style></style>