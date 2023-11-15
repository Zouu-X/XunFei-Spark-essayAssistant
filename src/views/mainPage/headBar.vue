<template>
  <div class="head-bar">
    <el-button type="text" @click="dialogVisible = true">Contact Me</el-button>
    <el-dialog
        title="Contact Me"
        :visible.sync="dialogVisible"
        width="30%"
    >
      <div class="contact-items">
        <span>Zoe</span>
        <span>2020210120</span>
        <span>Github:https://github.com/Zouu-X</span>
      </div>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
  </span>
    </el-dialog>
    <span>Welcome</span>
    <el-button class="style-change" @click="newChatEvent">New Chat</el-button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dialogVisible: false,
    }
  },
  methods: {
    newChatEvent() {
      this.$emit('newChatEvent')
    },
  }
}
</script>
<style lang="scss" scoped>
.head-bar {
  //background-color: #223E82;
  margin: -1px;
  height: 70px;
  width: 100%;
  border-bottom: 1px solid #dcdfe6;
  display: flex;
  align-items: center;
  justify-content: space-between;
  column-gap: 80px;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1); /* 调整阴影参数根据需要 */
}

.contact-items {
  display: flex;
  row-gap: 4px;
  flex-direction: column;
}
</style>