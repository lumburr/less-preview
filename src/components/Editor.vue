<script lang="ts" setup>
import Codemirror from './codemirror/Codemirror.vue';
interface Props {
  value?: string
  theme?: string
  readOnly?: boolean
  store?: Object
}
const { value, theme, readOnly, store } = defineProps<Props>()
const emit = defineEmits(['updateCode'])
const inputValue = (localValue: string) => {
  emit('updateCode', localValue)
}
</script>
<template>
  <Codemirror
    :value='value'
    :theme='theme'
    :readOnly='readOnly'
    @input="inputValue"/>
</template>