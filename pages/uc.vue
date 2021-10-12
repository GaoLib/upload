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
    <div class="progress">
      <p>计算hash进度</p>
      <el-progress :stroke-width="20" :text-inside="true" :percentage="hashProgress" />
    </div>
    <div>
      <div :style="{ width: `${cubeWidth}px` }" class="cube-container">
        <div v-for="chunk in chunks" :key="chunk.name" class="cube">
          <div
            :class="{
              'uploading': chunk.progress > 0 && chunk.progress < 100,
              'success': chunk.progress === 100,
              'error': chunk.progress < 0
            }"
          >
            <i v-if="chunk.progress<100 && chunk.progress > 0" class="el-icon-loading" style="color: #f56c6c" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import spartMD5 from 'spark-md5'

const CHUNK_SIZE = 0.1 * 1024 * 1024

export default {
  data() {
    return {
      file: null,
      // uploadProgress: 0,
      hashProgress: 0,
      hash: null,
      chunks: []
    }
  },
  computed: {
    cubeWidth() {
      return Math.ceil(Math.sqrt(this.chunks.length)) * 16
    },
    uploadProgress() {
      if (!this.file || this.chunks.length) {
        return 0
      }
      const loaded = this.chunks.map(item => item.chunk.size * item.progress)
        .reduce((acc, cur) => acc + cur, 0)
      return Number(((loaded * 100) / this.file.size).toFixed(2))
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
        this.hashProgress = 0
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
    createFileChunk(file, size = CHUNK_SIZE) {
      const chunks = []
      let cur = 0
      while (cur < file.size) {
        chunks.push({ index: cur, file: this.file.slice(cur, cur + size) })
        cur += size
      }
      return chunks
    },
    calculateHashWorker() {
      return new Promise((resolve) => {
        this.worker = new Worker('/hash.js')
        this.worker.postMessage({ chunks: this.chunks })
        this.worker.onmessage = (e) => {
          const { progress, hash } = e.data
          this.hashProgress = Number(progress.toFixed(2))
          if (hash) {
            resolve(hash)
          }
        }
      })
    },
    calculateHashIdle() {
      const chunks = this.chunks
      return new Promise((resolve) => {
        const spark = new spartMD5.ArrayBuffer()
        let count = 0

        const appendToSpark = (file) => {
          return new Promise((resolve) => {
            const reader = new FileReader()
            reader.readAsArrayBuffer(file)
            reader.onload = (e) => {
              spark.append(e.target.result)
              resolve()
            }
          })
        }

        const workLoop = async (deadline) => {
          while (count < chunks.length && deadline.timeRemaining() > 1) {
            await appendToSpark(chunks[count].file)
            count++
            if (count < chunks.length) {
              this.hashProgress = Number((100 * count / chunks.length).toFixed(2))
            } else {
              this.hashProgress = 100
              resolve(spark.end())
            }
          }
          window.requestIdleCallback(workLoop)
        }
        window.requestIdleCallback(workLoop)
      })
    },
    // ! 可用来进行预判断，布隆过滤器原理
    // ! hash 一样，文件不一定一样
    // ! hash 不一样， 文件一定不一样
    calculateHashSample() {
      return new Promise((resolve) => {
        const spark = new spartMD5.ArrayBuffer()
        const reader = new FileReader()

        const file = this.file
        const size = file.size
        const offset = 2 * 1024 * 1024
        const chunks = [file.slice(0, offset)]
        let cur = offset
        while (cur < size) {
          if (cur + offset >= size) {
            chunks.push(file.slice(cur, cur + offset))
          } else {
            const mid = cur + offset / 2
            const end = cur + offset
            chunks.push(file.slice(cur, cur + 2))
            chunks.push(file.slice(mid, mid + 2))
            chunks.push(file.slice(end - 2, end))
          }
          cur += offset
        }
        reader.readAsArrayBuffer(new Blob(chunks))
        reader.onload = (e) => {
          spark.append(e.target.result)
          this.hashProgress = 100
          resolve(spark.end())
        }
      })
    },
    async uploadFile() {
      // ! 图片格式校验
      // if (!await this.isImage(this.file)) {
      //   this.$message.error('文件格式错误')
      //   return
      // }
      const chunks = this.createFileChunk(this.file)
      // ! webworker
      // this.chunks = this.createFileChunk(this.file)
      // const hash = await this.calculateHashWorker()
      // ! idle
      // const hash1 = await this.calculateHashIdle()
      // ! 抽样hash
      this.hash = await this.calculateHashSample()
      this.chunks = chunks.map((chunk, index) => {
        const name = `${this.hash}-${index}`
        return {
          hash: this.hash,
          name,
          index,
          file: chunk.file
        }
      })
      // await this.uploadChunks()
      // ! 直接上传整个文件
      // const form = new FormData()
      // form.append('name', 'file')
      // form.append('file', this.file)
      // await this.$http.post('uploadfile', form, {
      //   onUploadProgress: (progress) => {
      //     this.uploadProgress = Number((progress.loaded / progress.total) * 100).toFixed(2)
      //   }
      // })
    },
    async uploadChunks() {
      const requests = this.chunks.map((chunk, index) => {
        const form = new FormData()
        form.append('chunk', chunk.chunk)
        form.append('hash', chunk.hash)
        form.append('name', chunk.name)
        return form
      }).map((form, index) => this.$http.post('/uploadfile', {
        onUploadProgress: (progress) => {
          this.chunks[index].progress = Number((progress.loaded / progress.total) * 100).toFixed(2)
        }
      }))
      await Promise.all(requests)
    }
  }
}
</script>

<style lang="stylus" scoped>
.drag
  width 300px
  height 100px
  border 2px dashed #eee
  text-align center
  line-height 100px
.cube-container
  .cube
    width 14px
    height 14px
    line-height 12px
    border 1px black solid
    background #eee
    float left
    >.suceess
      background green
    >.uploading
      background blue
    >.error
      background red
.progress
  width 300px
</style>
