<template>
  <Header @updateVue="updateVue" :store="store" />
  <section :class="{container: true, 'has-error': errorMessage, 'refresh': updateStyle}">
    <editor class="editor" v-model:value="input"/>
    <editor class="editor" v-model:value="output" readOnly />
  </section>
  <div v-if="errorMessage" class="error">
    {{errorMessage}}
  </div>
</template>

<script lang="ts">
import { nextTick } from 'vue'
import Header from './Header.vue'
import { utoa, atou } from './utils'
import Editor from '@components/Editor.vue'

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
      updateWithLessInterval,
      hash:'',
      init:false,
      store:{}
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
        this.serialize()
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
    serialize() {
      const newHash = '#' + utoa(JSON.stringify({
        code:this.input,
        activeVersion:this.store.activeVersion
      }))
      history.replaceState({}, '', newHash)
    }
  },
  components: {
    editor: Editor,
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
      if(!this.init && this.hash!=='' ) {
        this.serialize()
      }
      this.init = false
    },
  },
  created: async function() {
    this.hash = location.hash.slice(1)
    if(this.hash) {
      const saved = JSON.parse(atou(this.hash))
      this.init = true
      this.input = saved.code
      this.store = saved
    } else {
      this.serialize()
    }
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
  display: inline-block;
  margin: auto;
  width: calc(50vw - 12px);
  border: 1px solid hsl(190, 10%, 50%);
}
</style>
