<template>
  <div ref="list" @scroll="scrollEvent($event)" class="list-container">
    <div :style="{ height: `${listHeight}px`}" class="list-phantom" />
    <div :style="{ top: listTop }" class="list" />
  </div>
</template>

<script>
export default {
  props: {
    listData: {
      type: Array,
      default: () => []
    },
    size: {
      type: Number,
      default: 200
    }
  },
  data() {
    return {
      screenHeight: 800,
      startOffset: 0,
      start: 0,
      end: 4
    }
  },
  computed: {
    listHeight() {
      return this.listData.length * this.size
    },
    listTop() {
      return `${this.startOffset}px`
    },
    visibleCount() {
      return Math.ceil(this.screenHeight / this.size)
    },
    visibleData() {
      return this.listData.slice(this.start, Math.min(this.end, this.listData.length))
    }
  },
  mounted() {
    this.end = this.start + this.visibleCount
  },
  methods: {
    scrollEvent() {
      const scrollTop = this.$refs.list.scrollTop
      this.start = Math.floor(scrollTop / this.size)
      this.end = this.start + this.visibleCount
      this.startOffset = scrollTop - (scrollTop % this.size)
    }
  }
}
</script>

<style lang="stylus" scoped>
.list-container
  height 100%
  overflow hidden
  position relative
  .list-phantom
    position relative
    left 0
    top 0
    right 0
    z-index -1
  .list
    position relative
    left 0
    top 0
    right 0
</style>
