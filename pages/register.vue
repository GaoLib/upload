<template>
  <div class="login-container">
    <el-form ref="registerForm" :model="form" :rules="rules" class="login-form" label-width="100px">
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

      <el-form-item prop="nickname" label="昵称">
        <el-input v-model="form.nickname" placeholder="请输入昵称" />
      </el-form-item>

      <el-form-item prop="passwd" label="密码">
        <el-input v-model="form.passwd" type="password" placeholder="请输入密码" />
      </el-form-item>

      <el-form-item prop="repasswd" label="确认密码">
        <el-input v-model="form.repasswd" type="password" placeholder="请再次输入密码" />
      </el-form-item>

      <el-form-item label=" ">
        <el-button @click.native.prevent="handleRegister" type="primary">
          注册
        </el-button>
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
      form: {
        email: 'xingaoemail@163.com',
        nickname: 'Gao',
        passwd: '',
        repasswd: '',
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
        nickname: [
          { required: true, message: '请输入昵称' }
        ],
        passwd: [
          { required: true, pattern: /^[\w_-]{6,12}$/g, message: '请输入6~12位密码' }
        ],
        repasswd: [
          { required: true, message: '请再次输入密码' },
          { validator: (rule, value, callback) => {
            if (value !== this.form.passwd) {
              callback(new Error('两次密码不一致'))
            }
            callback()
          } }
        ]
      },
      code: {
        captcha: '/api/captcha'
      }
    }
  },
  methods: {
    resetCaptcha() {
      this.code.captcha = '/api/captcha?_t' + new Date().getTime()
    },
    handleRegister() {
      this.$refs.registerForm.validate(async (valid) => {
        if (valid) {
          const obj = {
            email: this.form.email,
            nickname: this.form.nickname,
            passwd: md5(this.form.passwd),
            captcha: this.form.captcha
          }
          const ret = await this.$http.post('/user/register', obj)
          if (ret.code === 0) {
            this.$alert('注册成功', '成功', {
              confirmButtonText: '去登录',
              callback: () => {
                this.$router.push('/login')
              }
            })
          } else {
            this.$message.error(ret.message)
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
