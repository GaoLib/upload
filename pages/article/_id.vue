<template>
  <div class="container">
    <UserDisplay :user="article.author">
      <el-button v-if="isFollow" type="success" @click="cancelfollow">已关注</el-button>
      <el-button v-else @click="follow">关注</el-button>
    </UserDisplay>

    <el-divider />
    <div class="article" v-html="article.article_html">

    </div>
    <el-divider />
    <el-button @click="likeAction" :type="likeStatus ? 'success' : 'default'">
      <i class="el-icon-thumb">{{ article.like }}</i>
    </el-button>
  </div>
</template>

<script>
import UserDisplay from '~/components/UserDisplay.vue'

export default {
  components:{ UserDisplay },
  data(){
    return {
      isFollow: false,
      likeStatus: false,
      dislikeStatus: false,
      article:{
        title: '',
        author: {}
      }
    }
  },
  mounted(){
    let { id } = this.$route.params
    this.id = id
    this.getArticle()

    // 用户已登录 
    const token = localStorage.getItem('USER_TOKEN')
    this.token = token
  },
}
</script>

<style lang="stylus" scoped>
.article
  padding 10px
.rotate
  transform rotate(180deg)
</style>
