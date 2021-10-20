<template>
  <div>
    <div class="write-btn">
      <el-button @click="submit" type="primary">
        提交
      </el-button>
    </div>
    <el-row>
      <el-col :span="12">
        <textarea :valve="content" @input="update" cols="30" rows="10" class="md-editor" />
      </el-col>
      <el-col :span="12">
        <div v-html="compiledContent" class="markdown-body" />
      </el-col>
    </el-row>
  </div>
</template>

<script>
import marked from 'marked'

export default {
  data() {
    return {
      content: `# 学习吖
      * 上课
      * 看书
      * 工作`
    }
  },
  computed: {
    compiledContent() {
      return marked(this.content, {})
    }
  },
  methods: {
    update(e) {
      this.content = e.target.value
    },
    submit() {}
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
