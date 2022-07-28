<script lang="ts" setup>
  import Loading from '@components/Loading.vue'
  import {toRefs} from 'vue';
  const props = defineProps(["loadingLessJS"]);
  let { loadingLessJS } = toRefs(props);
</script>
<template>
  <section class="container">
    <body class="body">
      <div class="left" style="width: 50%;">
        <div class="fileSelector">
          <slot name="file-selector"></slot>
        </div>
        <div class="editor">
          <slot name="edit"></slot>
        </div>
      </div>      
      <div class="right" style="width: 50%;">
        <div class="editor">
          <slot name="preview"></slot>
        </div>
      </div> 
    </body>
    <footer class="footer">
      <slot name="footer"></slot>
    </footer>
    <Loading v-show="loadingLessJS"></Loading>
  </section>
</template>
<style lang="less">
.container {
  position: relative;
  height: calc(100vh - 40px);
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  padding: 6px 0;
}
.body {
  overflow: hidden;
  display: flex;
  justify-content: space-evenly;
  align-items: stretch;
  flex: 1;
}

.editor {
  width: calc(50vw - 12px);
  border: 1px solid hsl(190, 10%, 50%);
}

.footer {
  width: 100%;
}
</style>