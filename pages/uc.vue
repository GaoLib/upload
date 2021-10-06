<template>
  <div>
    <h1>用户中心</h1>
    <div ref="drag" class="drag">
      <input @change="handleFileChanged" type="file" name="file"></input>
    </div>
    <div class="progress">
      <el-progress :stroke-width="20" :text-inside="true" :percentage="uploadProgress" />
    </div>
    <div>
      <el-button @click="uploadFile">
        上传
      </el-button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      file: null,
      uploadProgress: 0
    }
  },
  async mounted() {
    await this.$http.get('/user/info')
    this.bindEvents()
  },
  methods: {
    bindEvents() {
      const drag = this.$refs.drag
      drag.addEventListener('dragover', (e) => {
        drag.style.borderColor = 'red'
        e.preventDefault()
      })
      drag.addEventListener('dragleave', (e) => {
        drag.style.borderColor = '#eee'
        e.preventDefault()
      })
      drag.addEventListener('drop', (e) => {
        const fileList = e.dataTransfer.files
        drag.style.borderColor = '#eee'
        this.file = fileList[0]
        e.preventDefault()
      })
    },
    handleFileChanged(e) {
      const file = e.target.files[0]
      if (file) {
        this.file = file
      }
    },
    async uploadFile() {
      const form = new FormData()
      form.append('name', 'file')
      form.append('file', this.file)
      await this.$http.post('uploadfile', form, {
        onUploadProgress: (progress) => {
          this.uploadProgress = Number((progress.loaded / progress.total) * 100).toFixed(2)
        }
      })
    }
  }
}
</script>

<style lang="stylus" scoped>
.drag
  width 200px
  height 100px
  border 2px dashed #eee
  text-align center
  line-height 100px
.progress
  width 200px
</style>
