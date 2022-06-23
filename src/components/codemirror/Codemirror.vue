<template>
  <div class="editor" ref="el"/>
</template>
<script setup lang="ts">
import { onMounted } from 'vue'
import { basicSetup, EditorView } from "codemirror"
import { css } from '@codemirror/lang-css'

interface Props {
  value?: string
  theme?: string
  readOnly?
  
  
  : boolean
}

const el = $ref()
const { value, theme, readOnly = false } = defineProps<Props>()

onMounted(() => {
  new EditorView({
    doc: value,
    extensions: [basicSetup, css()],
    parent: el as Element
  })
})
</script>
