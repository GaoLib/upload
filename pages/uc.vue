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
    blobToString(blob) {
      return new Promise((resolve) => {
        const reader = new FileReader()
        reader.onload = () => {
          const ret = reader.result.split('')
            .map(v => v.charCodeAt())
            .map(v => v.toString(16).toUpperCase())
            .map(v => v.padStart(2, '0'))
            .join('')
          resolve(ret)
        }
        reader.readAsBinaryString(blob)
      })
    },
    async isGif(file) {
      // '47 49 46 38 39 61' || '47 49 46 38 37 61'
      const ret = await this.blobToString(file.slice(0, 6))
      const string = ret.replace(/\s/g, '')
      return (string === '474946383961') || (string === '474946383761')
    },
    async isPng(file) {
      // '89 50 4E 47 0D 0A 1A 0A'
      const ret = await this.blobToString(file.slice(0, 8))
      const string = ret.replace(/\s/g, '')
      return string === '89504E470D0A1A0A'
    },
    async isJpg(file) {
      const len = file.size
      const start = await this.blobToString(file.slice(0, 2))
      const tail = await this.blobToString(file.slice(-2, len))
      return start.replace(/\s/g, '') === 'FFD8' && tail.replace(/\s/g, '') === 'FFD9'
    },
    async isImage(file) {
      const isGif = await this.isGif(file)
      const isPng = await this.isPng(file)
      const isJpg = await this.isJpg(file)
      return isGif || isPng || isJpg
    },
    async uploadFile() {
      if (!await this.isImage(this.file)) {
        this.$message.error('文件格式错误')
        return
      }
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
