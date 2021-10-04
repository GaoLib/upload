<template>
  <div>
    <h1>用户中心</h1>
    <div>
      <input type="file" @change="handleFileChanged" name="file" />
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
      file: null
    }
  },
  async mounted() {
    const ret = await this.$http.get('/user/info')
    console.log(ret)
  },
  methods: {
    handleFileChanged(e) {
      const file = e.target.files
      if (!file) return
      this.file = file
    },
    async uploadFile() {
      const form = new FormData()
      form.append('name', 'file')
      form.append('file', this.file)
      const ret = await this.$http.post('uploadfile', form)
      console.log(ret)
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
