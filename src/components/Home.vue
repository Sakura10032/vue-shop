<template>
  <div>
    Home
    <el-button @click="logout" type="info">退出</el-button>
  </div>
</template>

<script>
  export default {
    methods: {
      logout () {
        window.sessionStorage.clear()
        this.$message.success('退出成功！')
        this.$router.push('/login')
      }
    }
  }
</script>

<style lang="less" scoped>

</style>
