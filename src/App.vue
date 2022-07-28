<script lang="ts" setup>
import { nextTick, onMounted, reactive, watch } from 'vue'
import { utoa, atou, LESS_DATA } from './utils'
import Header from '@components/Header.vue'
import FileSelector from '@components/FileSelector.vue'
import Editor from '@components/Editor.vue'
import Layout from '@components/Layout.vue'
import postcss from 'postcss';
import syntax from 'postcss-less';
interface File {
  code: string
  filename: string
}
interface Store {
  activeVersion: string
  mainFile: string
  activeFile: File
  files: Record<string, File>
}
let output = $ref<string | undefined>('')
let errorMessage = $ref('')
let hash = $ref('')
let key = $ref(1)
let activeFileCode = $ref(LESS_DATA)
let store: Store = reactive(
  { 
    activeVersion: '4.x',
    mainFile: 'index.less',
    activeFile:{
      code: LESS_DATA,
      filename: 'index.less'
    },
    files: {  
      'index.less': {
        code: LESS_DATA,
        filename: 'index.less'
      }
    },
  }
)
let loadingLessJS = $ref(false)

const serialize = () => {
  const newHash = '#' + utoa(JSON.stringify(store))
  history.replaceState({}, '', newHash)
}
const updateCode = (code: string)=>{
  store.files[store.activeFile.filename as keyof typeof store.files].code = code
  store.activeFile.code = code
  activeFileCode = code
  updateOutput()
  if (hash !== '') {
    serialize()
  }
}

const updateOutput = () => {
  let astRsults:any = []
  let fileList = Object.keys(store.files)
  fileList.forEach((file)=>{
    postcss()
    .process(store.files[file].code, { syntax: syntax })
    .then(function (result) {
      astRsults.push(result.root)
      if(astRsults.length === fileList.length ){
        let finalAst = replaceAstNode(astRsults, fileList, 0)
        postcss()
        .process(finalAst, { syntax: syntax })
        .then(function (finalResult) {
          parseLess(finalResult.css)
        });
      }
    });
  })
}

const upLoadingLessJS = (loadingFlag:boolean) =>{
  loadingLessJS = loadingFlag
}
const replaceAstNode = (astRsults: string | any[], fileList: string | any[], astRsultIndex:number)=>{
  let length = astRsults.length
  if(astRsultIndex>=length || astRsultIndex === -1){
    return false
  }
  astRsults[astRsultIndex].walk((node: any) => {
    console.log('beforeNode')
    console.log(node)
    if(node.type === 'atrule' && node.name==='import') {
      node = replaceAstNode(astRsults, fileList, fileList.indexOf(node.params))
    }
  });
  return astRsults[astRsultIndex]
}
const parseLess = (less: string) =>{
  window.less.render(less, {}, (error, result) => {
    if (error) {
      errorMessage = error.message
      output = ''
    } else {
      errorMessage = ''
      output = result?.css
    }
  })
}

hash = location.hash.slice(1)
if (hash) {
  const saved = JSON.parse(atou(hash))
  store = saved
} else {
  store.activeFile.filename = 'index.less'
  store.activeFile.code = LESS_DATA
  serialize()
}

watch(() => activeFileCode, () => {
  updateOutput()
  if (hash !== '') {
    serialize()
  }
})
watch(() => activeFileCode, (activeFileCode)=>{
  store.files[store.activeFile.filename as keyof typeof store.files].code = activeFileCode
  store.activeFile.code = activeFileCode
})

//  "addFile", "setActive", "deleteFile" 
const setActive = (file:string) => {
  store.activeFile = store.files[file as keyof typeof store.files]
  activeFileCode = store.activeFile.code
  key++
}
const addFile = (file:string) => {

  if(store.files[file as keyof typeof store.files]===undefined){
    store.files[file as keyof typeof store.files] = {
      code:'',
      filename: file
    }
    store.activeFile.code = ''
    store.activeFile.filename = file
    activeFileCode = store.activeFile.code
    key++
  }
}
const deleteFile = (file:string) => {
  if(store.files[file as keyof typeof store.files]!==undefined){
    if (store.activeFile.filename === file) {
      store.activeFile = store.files[store.mainFile as keyof typeof store.files]
      activeFileCode = store.activeFile.code
    }
    delete store.files[file as keyof typeof store.files]
  }
}

onMounted(() => {
  nextTick(() => {
    updateOutput()
  })
})

</script>
<template>
  <Header @updateOutput="updateOutput" :store="store" @upLoadingLessJS="upLoadingLessJS"/>
  <Layout :loadingLessJS="loadingLessJS">
    <template #file-selector>
      <!-- "addFile", "setActive", "deleteFile" -->
      <FileSelector :store="store" @setActive="setActive" @addFile="addFile" @deleteFile="deleteFile"/>
    </template>
    <template #edit>
      <editor v-model:value="activeFileCode" :store="store" @updateCode="updateCode" :key="key"/>
    </template>
    <template #preview>
      <editor v-model:value="output" readOnly />
    </template>
    <template #footer>
      <div v-if="errorMessage" class="error">
        {{ errorMessage }}
      </div>
    </template>
  </Layout>
</template>
<style lang="less">
html,
body {
  margin: 0;
}

@base: #35495e;

body {
  background: @base;
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

@keyframes opac {
  0% {
    opacity: 0;
  }

  100% {
    opacity: 1;
  }
}

.error {
  color: hsl(10, 89%, 78%);
  background: hsl(10, 89%, 26%);
  height: 28px;
  padding: 2px 8px;
  position: relative;
  top: -6px;
  animation: opac 1s;
}
</style>
