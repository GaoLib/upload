<template>
  <div>
    <div class="write-btn">
      <el-button @click="submit" type="primary">
        提交
      </el-button>
    </div>
    <el-row>
      <el-col :span="12">
        <textarea
          ref="editor"
          :value="content"
          @input="update"
          cols="30"
          rows="10"
          class="md-editor"
        />
      </el-col>
      <el-col :span="12">
        <div v-html="compiledContent" class="markdown-body" />
      </el-col>
    </el-row>
  </div>
</template>

<script>
import marked from 'marked'
import hljs from 'highlight.js'
// eslint-disable-next-line no-unused-vars
import javascript from 'highlight.js/lib/languages/javascript'
import 'highlight.js/styles/monokai-sublime.css'

export default {
  data() {
    return {
      content: `# 学习吖
      * 上课
      * 看书
      * 工作
\`\`\`javascript
  let a =1;
  console.log(a)
\`\`\`
      `
    }
  },
  computed: {
    compiledContent() {
      return marked(this.content, {})
    }
  },
  mounted() {
    this.timer = null
    this.bindEvent()

    marked.setOptions({
      rendered: new marked.Renderer(),
      highlight(code) {
        return hljs.highlightAuto(code).value
      }
    })
  },
  methods: {
    bindEvent() {
      this.$refs.editor.addEventListener('paste', (e) => {
        // const files = e.clipboardData.files
      })
      this.$refs.editor.addEventListener('drop', (e) => {
        // const files = e.dataTransfer.files
        e.preventDefault()
      })
    },
    update(e) {
      clearTimeout(this.timer)
      this.timer = setTimeout(() => {
        this.content = e.target.value
      }, 350)
    },
    async submit() {
      await this.$http.post('/article/create', {
        content: this.content,
        compiledContent: this.compiledContent
      })
    }
  }
}
</script>

<style lang="stylus" scoped>
.md-editor
  width 100%
  height 100vh
  outline none
.write-btn
  position fixed
  z-index 100
  right 30px
  top 10px
</style>
