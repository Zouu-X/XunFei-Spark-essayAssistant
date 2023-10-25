<template>
  <div>
    <div class="head-bar">
      <el-button type="text" @click="open">Contact Me</el-button>
      <span>Welcome</span>
      <el-button class="style-change">Model change</el-button>
    </div>
    <div class="main-body">
      <div class="chat-container">

      </div>
    </div>
  </div>

</template>

<script>
export default {
  name: "index",
  data() {
    return {}
  },
  methods: {
    // open() {
    //   this.$alert
    // }
  }
}
</script>
<style scoped>
.head-bar {
  height: 70px;
  width: 100%;
  border-bottom: 1px solid #dcdfe6;
  display: flex;
  .style-change {
    right: 0px;
  }
}
.main-body {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
  .chat-container {
    height: 300px;
    width: 600px;
    border: 1px solid black;
  }

}
</style>
