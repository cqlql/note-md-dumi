---
nav: Vue3
title: 双向绑定实现
---

组件内双向绑定实现方案，涉及多级 modelValue 嵌套

## 1. 直接使用 props.modelValue

```vue
<script lang="ts" setup>
const props = defineProps<{
  modelValue: string;
}>();

const emit = defineEmits<{
  'update:modelValue': [v: string];
}>();

function onInput(v: string) {
  emit('update:modelValue', v);
}
</script>

<template>
  <div class="InputDemo">
    <el-input :model-value="modelValue" @input="onInput" />
  </div>
</template>
```

## 2. 通过 watchEffect

适用组件内已单独声明值的情况，属于更灵活的方式，方便后期扩展

实时改变的例子：

```vue
<script lang="ts" setup>
const props = defineProps<{
  modelValue: string;
}>();

const value = ref('');

watchEffect(() => {
  value.value = props.modelValue;
});

const emit = defineEmits<{
  'update:modelValue': [v: string];
}>();

function onInput(v: string) {
  emit('update:modelValue', v);
}
</script>

<template>
  <div class="InputDemo">
    <el-input :model-value="modelValue" @input="onInput" />
  </div>
</template>
```

带确认修改的例子：

```vue
<script lang="ts" setup>
const props = defineProps<{
  modelValue: string;
}>();

const value = ref('');

watchEffect(() => {
  value.value = props.modelValue;
});

const emit = defineEmits<{
  'update:modelValue': [v: string];
}>();

function confirm() {
  emit('update:modelValue', value.value);
}
</script>

<template>
  <div class="InputDemo">
    <el-input v-model="value" />
    <button @click="confirm">确认修改</button>
  </div>
</template>
```
