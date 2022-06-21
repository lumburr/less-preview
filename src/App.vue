<template>
  <div>
    <Header @updateVue="updateVue"></Header>
    <section :class="{container: true, 'has-error': errorMessage, 'refresh': updateStyle}">
      <div class="editor">
        <editor
          v-model:value="input"
          lang="less"
          theme="crunch-dark"
          style="height: 100%"
          :options="{
            useWorker: true,
            useSoftTabs: true,
            tabSize: 2,
          }"
        ></editor>
      </div>
      <div class="editor">
        <editor
          v-model:value="output"
          @init="editorInit"
          lang="less"
          theme="crunch-dark"
          style="height: 100%"
          :options="{
            useSoftTabs: true,
            tabSize: 2,
            readOnly: true
          }"
        ></editor>
      </div>
    </section>
    <div v-if="errorMessage" class="error">
      {{errorMessage}}
    </div>
  </div>
</template>

<script>
import { nextTick } from 'vue'
import Header from './Header.vue'
import { VAceEditor } from 'vue3-ace-editor';
import 'ace-builds/src-noconflict/ext-language_tools';
import 'ace-builds/src-noconflict/mode-less';
import './theme-crunch-dark';

export default {
  data() {
    let updateWithLessInterval
    return {
      updateStyle: false,
      input: 
`#lib() {
  .colors() {
    @primary: blue;
    @secondary: green;
  }
  .rules(@size) {
    border: @size solid white;
  }
}

.box when (#lib.colors[@primary] = blue) {
  width: 100px;
  height: ($width / 2);
}

.bar:extend(.box) {
  @media (min-width: 600px) {
    width: 200px;
    #lib.rules(1px);
  }
}`,
      output: '',
      errorMessage: '',
      updateWithLessInterval
    }
  },
  methods: {
    updateVue() {
      if (!window.less) {
        this.updateWithLessInterval = setInterval(this.updateWithLess, 100)
      } else {
        this.updateWithLess()
      }
    },
    updateWithLess() {
      if (window.less) {
        clearInterval(this.updateWithLessInterval)
        const self = this
        window.less.render(this.input, {}, function(error, output) {
          if (error) {
            self.errorMessage = error.message
            self.output = ''
          } else {
            self.errorMessage = ''
            self.output = output.css
          }
        })
      }
    },
    editorInit: function () {
    },
  },
  components: {
    editor: VAceEditor,
    Header
  },
  mounted() {
    nextTick(() => {
      this.updateVue(this.input)
      this.updateStyle = true
      requestAnimationFrame(() => {
        void document.offsetHeight
        this.updateStyle = false
      })
    })
  },
  watch: {
    input(newInput) {
      this.updateVue(newInput)
    },
  },
  created: async function() {
  },
}
</script>

<style lang="less">
html, body {
  margin: 0;
}
@base: #35495e;
body {
  background: @base;
}
*, *::before, *::after {
  box-sizing: border-box;
}

.container {
  height: calc(100vh - 40px);
  display: flex;
  justify-content: space-evenly;
  align-items: stretch;
  text-align: center;
  padding: 6px 0;
  &.has-error {
    height: calc(100vh - 60px);
  }
  &.refresh {
    text-indent: 0.1px;
    -webkit-text-stroke: 0.1px transparent;
  }
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
.editor {
  width: calc(50vw - 12px);
  border: 1px solid hsl(190, 10%, 50%);
}
</style>
