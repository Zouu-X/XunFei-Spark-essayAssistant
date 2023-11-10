<template>
  <div>
    <div class="head-bar">
      <el-button type="text" @click="open">Contact Me</el-button>
      <span>Welcome</span>
      <el-button class="style-change">Model change</el-button>
    </div>
    <div class="main-body">
      <div class="chat-container">
        <div class="chat-messages">
          <div
            v-for="(message, index) in messages"
            :key="index"
            class="msg-group"
          >
            <div class="msg-head">
              {{ message.isMine ? 'User' : 'AI' }}
            </div>
            <div
              class="message"
              :class="{ 'my-message' : message.isMine}"
            >
              {{ message.text }}
            </div>
          </div>
        </div>
      </div>
      <el-input
      style="width: 650px; margin-top: 20px;"
      type="text"
      placeholder="请输入内容"
      v-model="textInput"
      @keyup.enter.native="Submit"
      >
        <el-button slot="append" icon="el-icon-thumb" @click="Submit" />
      </el-input>

      <transition name="fade">
        <el-alert
          v-if="showClosingAlert"
          title="无效内容"
          type="error"
          effect="light"
          show-icon
        />
      </transition>
      <el-radio-group
          v-model="radio"
          @input="inputListener"
          style="margin-top: 30px;"
      >
        <el-radio-button label="TEST1"></el-radio-button>
        <el-radio-button label="TEST2"></el-radio-button>
        <el-radio-button label="TEST3"></el-radio-button>
        <el-radio-button label="TEST4"></el-radio-button>
      </el-radio-group>
    </div>
  </div>

</template>

<script>
export default {
  name: "index",
  data() {
    return {
      messages: [],
      showClosingAlert: false,
      textInput: '',
      radio: 'TEST1'

    }
  },

  methods: {
    open() {
      this.$alert("Zoe", "联系方式", {confirmButtonText: '确定'})
    },
    inputListener() {
      // console.log(this.radio)
    },
    Submit() {
      if (this.textInput === '') {
        this.showClosingAlert = true
        setTimeout(() => {
          this.showClosingAlert = false
        }, 2000)
      }else {
        this.messages.push({ text: this.textInput, isMine: true })
        setTimeout(() => {
          this.messages.push({ text: "test", isMine: false})
        }, 1000)
        this.textInput = ''
      }
    }
  }
}
</script>
<style lang="scss" scoped>
.head-bar {
  //background-color: #223E82;
  height: 70px;
  width: 100%;
  border-bottom: 1px solid #dcdfe6;
  display: flex;
  align-items: center;
  column-gap: 80px;

}
.main-body {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
  .el-alert {
    width: 400px;
  }
}
.chat-container {
  display: flex;
  flex-direction: column;
  height: 300px;
  width: 80%;
  border: 1px solid #ccc;
  border-radius: 5px;
  overflow: hidden;
}
.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
}
.msg-group {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
}
.msg-head {
  text-align: center; /* 文字水平居中 */
  line-height: 36px; /* 文字垂直居中，与容器高度相等 */
  width: 36px;
  height: 36px;
  padding: 6px;
  margin: 5px;
  border-radius: 5px;
  background: yellowgreen;
}
.message {
  min-height: 36px;
  width: 100%;
  background-color: #CB9836;
  padding: 6px;
  margin: 5px;
  border-radius: 5px;
  display: flex;
}

.my-message {
  background-color: #223E82;
  color: white;
}

.chat-input {
  display: flex;
  align-items: center;
  padding: 10px;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
