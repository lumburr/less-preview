<template>
  <Codemirror :value='value' :theme='theme' :readOnly='readOnly'/>
</template>
<script lang="ts" setup>
import Codemirror from './codemirror/Codemirror.vue';
interface Props {
  value?: string
  theme?: string
  readOnly?: boolean
}
const { value, theme, readOnly } = defineProps<Props>()
</script>
<style lang="less">
</style>