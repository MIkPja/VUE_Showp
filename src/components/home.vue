<template>
  <div>
    <el-button type='info'>退出</el-button>
  </div>
</template>

<script>
export default {
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/')
    }
  }
}
</script>

<style lang="less" scoped>

</style>
