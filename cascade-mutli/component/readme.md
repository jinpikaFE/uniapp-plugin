# cascade-mutli

基于 https://ext.dcloud.net.cn/plugin?id=14321 插件开发二次开发；并修复相关问题

## 添加属性

- isMulti: Boolean 最后一列是否支持多选
- 完善回显效果

## 使用方法

```vue
<template>
  <cascade-mutli
    inputAlign="right"
    placeholder=""
    isMulti
    v-model="formModel.smoking"
    @confirm="onSmkConfirm"
    :options="multiSmkPickerList"
  />
</template>

<script lang="ts" setup>
const formModel = reactive<{
  smoking: any[];
}>({
  smoking: [0, 1, 3], // [0, 1, '1,2,3'] 多选
});
const num99Arr = Array.from({ length: 99 }, (_, i) => i + 1);

const multiSmkPickerList = ref([
  {
    label: "吸烟",
    value: 1,
    children: num99Arr?.map((item) => ({
      label: `${item}年烟龄`,
      value: item,
      children: num99Arr?.map((item) => ({
        label: `${item}支/天`,
        value: item,
      })),
    })),
  },
  { label: "不吸烟", value: 0 },
]);
</script>
```
