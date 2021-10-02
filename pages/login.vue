<template>
  <div class="login-container">
    <el-form ref="loginForm" :model="form" :rules="rules" label-width="100px" class="login-form">
      <!-- <div class="title-container">
        <img src="/logo.png" alt="">
      </div> -->
      <el-form-item prop="email" label="邮箱">
        <el-input v-model="form.email" placeholder="请输入邮箱" />
      </el-form-item>
      <el-form-item prop="captcha" label="验证码" class="captcha-container">
        <div class="captcha">
          <img :src="code.captcha" @click="resetCaptcha">
        </div>
        <el-input v-model="form.captcha" placeholder="请输入验证码" />
      </el-form-item>

      <el-form-item prop="emailcode" label="验证码" class="captcha-container">
        <div class="captcha">
          <el-button @click="sendEmailCode" :disabled="send.timer>0" type="primary">
            {{ sendText }}
          </el-button>
        </div>
        <el-input v-model="form.emailcode" placeholder="请输入邮件验证码" />
      </el-form-item>

      <el-form-item prop="passwd" label="密码">
        <el-input v-model="form.passwd" type="password" placeholder="请输入密码" />
      </el-form-item>

      <el-form-item label=" ">
        <el-button @click.native.prevent="handleLogin" type="primary">
          登录
        </el-button>
        <nuxt-link to="/register">
          <el-button type="primary">
            注册
          </el-button>
        </nuxt-link>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import md5 from 'md5'
export default {
  layout: 'login',
  data() {
    return {
      send: {
        timer: 0
      },
      form: {
        email: 'xingaoemail@163.com',
        passwd: '',
        captcha: ''
      },
      rules: {
        email: [
          { required: true, message: '请输入邮箱' },
          { type: 'email', message: '请输入正确的邮箱格式' }
        ],
        captcha: [
          { required: true, message: '请输入验证码' }
        ],
        emailcode: [
          { required: true, message: '请输入邮箱验证码' }
        ],
        passwd: [
          { required: true, pattern: /^[\w_-]{6,12}$/g, message: '请输入6~12位密码' }
        ]
      },
      code: {
        captcha: '/api/captcha'
      }
    }
  },
  computed: {
    sendText() {
      if (this.send.timer <= 0) {
        return '发送'
      }
      return `${this.send.timer}s后发送`
    }
  },
  methods: {
    async sendEmailCode() {
      await this.$http.get('/sendcode?email=' + this.form.email)
      this.send.timer = 60
      this.timer = setInterval(() => {
        this.send.timer -= 1
        if (this.send.timer === 0) {
          clearInterval(this.timer)
        }
      }, 1000)
    },
    resetCaptcha() {
      this.code.captcha = '/api/captcha?_t' + new Date().getTime()
    },
    handleLogin() {
      this.$refs.loginForm.validate(async (valid) => {
        if (valid) {
          const obj = {
            email: this.form.email,
            passwd: md5(this.form.passwd),
            captcha: this.form.captcha,
            emailcode: this.form.emailcode
          }
          const ret = await this.$http.post('/user/login', obj)
          if (ret.code === 0) {
            // token的存储 登录成功，返回token
            this.$message.success('登录成功')
            // localStorage.setItem('KKB_USER_TOKEN', ret.data.token)
            localStorage.setItem('token', ret.data.token)
            setTimeout(() => {
              this.$router.push('/')
            }, 500)
          } else {
            this.$message.error(ret.msg)
          }
        } else {
          this.$message.error('校验失败')
        }
      })
    }
  }
}
</script>

<style lang="stylus">

</style>
