<template></template>
<script setup lang="ts">
function setPropertyMap(propertyMap: Map<string, string>, property: string[], value: string[]): Map<string, string> {
    for (let i = 0; i < property.length; i++) {
        propertyMap.set(property[i], value[i])
    }
    return propertyMap
}
let propertyMap = new Map()
setPropertyMap(propertyMap,["1"],["123"])
console.log(propertyMap);

</script>
<style></style>