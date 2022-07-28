<script setup lang="ts">
import { computed, inject, ref, VNode, Ref } from 'vue'
const emit = defineEmits(["addFile", "setActive", "deleteFile"]);
const props = defineProps(["store"]);
const { store } = props;

const pending = ref(false)
const pendingFilename = ref('next.less')
const importMapFile = 'import-map.json'
const showImportMap = inject('import-map') as Ref<boolean>
const files = computed(() =>
  Object.entries(store.files)
    .filter(([name]) => name !== importMapFile)
    .map(([name]) => name)
)

function startAddFile() {
  let i = 0
  let name = `next.less`

  while (true) {
    let hasConflict = false
    for (const file in store.files) {
      if (file === name) {
        hasConflict = true
        name = `next${++i}.less`
        break
      }
    }
    if (!hasConflict) {
      break
    }
  }

  pendingFilename.value = name
  pending.value = true
}

function cancelAddFile() {
  pending.value = false
}

function focus({ el }: VNode) {
  ;(el as HTMLInputElement).focus()
}

function doneAddFile() {
  if (!pending.value) return
  const filename = pendingFilename.value

  if (!/\.(less|css)$/.test(filename)) {
    store.errors = [
      `Playground only supports *.less, *.css files.`
    ]
    return
  }

  if (filename in store.files) {
    store.errors = [`File "${filename}" already exists.`]
    return
  }

  store.errors = []
  cancelAddFile()
  debugger
  emit('addFile', filename)
}
function setActive(file: string) {
    emit('setActive', file)
}
function deleteFile(file: string) {
    emit('deleteFile', file)
}

const fileSel = ref(null)
function horizontalScroll(e: WheelEvent) {
  e.preventDefault()
  const el = fileSel.value! as HTMLElement
  const direction =
    Math.abs(e.deltaX) >= Math.abs(e.deltaY) ? e.deltaX : e.deltaY
  const distance = 30 * (direction > 0 ? 1 : -1)
  el.scrollTo({
    left: el.scrollLeft + distance
  })
}
</script>

<template>
  <div
    class="file-selector"
    :class="{ 'has-import-map': showImportMap }"
    @wheel="horizontalScroll"
    ref="fileSel"
  >
    <div
      v-for="(file, i) in files"
      class="file"
      :class="{ active: store.activeFile.filename === file }"
      @click="setActive(file)"
    >
      <span class="label">{{
        file === importMapFile ? 'Import Map' : file
      }}</span>
      <span v-if="i > 0" class="remove" @click.stop="deleteFile(file)">
        <svg class="icon" width="12" height="12" viewBox="0 0 24 24">
          <line stroke="#999" x1="18" y1="6" x2="6" y2="18"></line>
          <line stroke="#999" x1="6" y1="6" x2="18" y2="18"></line>
        </svg>
      </span>
    </div>
    <div v-if="pending" class="file pending">
      <input
        v-model="pendingFilename"
        spellcheck="false"
        @blur="doneAddFile"
        @keyup.enter="doneAddFile"
        @keyup.esc="cancelAddFile"
        @vnodeMounted="focus"
      />
    </div>
    <button class="add" @click="startAddFile">+</button>

    <div v-if="showImportMap" class="import-map-wrapper">
      <div
        class="file import-map"
        :class="{ active: store.activeFile.filename === importMapFile }"
        @click="setActive(importMapFile)"
      >
        <span class="label">Import Map</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.file-selector {
  display: flex;
  box-sizing: border-box;
  border-bottom: 1px solid var(--border);
  background-color: var(--bg);
  overflow-y: hidden;
  overflow-x: auto;
  white-space: nowrap;
  position: relative;
  height: var(--header-height);
}

.file-selector::-webkit-scrollbar {
  height: 1px;
}

.file-selector::-webkit-scrollbar-track {
  background-color: var(--border);
}

.file-selector::-webkit-scrollbar-thumb {
  background-color: #42d392;
}

.file-selector.has-import-map .add {
  margin-right: 10px;
}

.file {
  display: inline-block;
  font-size: 13px;
  font-family: var(--font-code);
  cursor: pointer;
  color: var(--text-light);
  box-sizing: border-box;
}
.file.active {
  color: #42d392;
  border-bottom: 3px solid #42d392;
  cursor: text;
}
.file span {
  display: inline-block;
  padding: 8px 10px 6px;
  line-height: 20px;
}
.file.pending input {
  width: 90px;
  height: 30px;
  line-height: 30px;
  outline: none;
  border: 1px solid var(--border);
  border-radius: 4px;
  padding: 0 0 0 10px;
  margin-top: 2px;
  margin-left: 6px;
  font-family: var(--font-code);
  font-size: 12px;
}
.file .remove {
  display: inline-block;
  vertical-align: middle;
  line-height: 12px;
  cursor: pointer;
  padding-left: 0;
}
.add {
  font-size: 18px;
  font-family: var(--font-code);
  color: #999;
  vertical-align: middle;
  margin-left: 6px;
  position: relative;
  top: -1px;
}
.add:hover {
  color: #42d392;
}
.icon {
  margin-top: -1px;
}
.import-map-wrapper {
  position: sticky;
  margin-left: auto;
  top: 0;
  right: 0;
  padding-left: 30px;
  background-color: var(--bg);
  background: linear-gradient(
    90deg,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 1) 25%
  );
}
.dark .import-map-wrapper {
  background: linear-gradient(
    90deg,
    rgba(26, 26, 26, 0) 0%,
    rgba(26, 26, 26, 1) 25%
  );
}
</style>
