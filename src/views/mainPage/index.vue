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
      <el-input
      style="width: 650px; margin-top: 20px;"
      type="text"
      placeholder="请输入内容"
      v-model="textInput"
      @change="Submit"
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
          style="margin-top: 20px;"
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
        console.log(this.textInput)
        this.textInput = ''
      }

    }
  }
}
</script>
<style lang="scss" scoped>
.head-bar {
  height: 70px;
  width: 100%;
  border-bottom: 1px solid #dcdfe6;
  display: flex;
  justify-content: center;
  align-items: center;
  column-gap: 80px;

}
.main-body {
  display: flex;
  flex-direction: column;
  //row-gap: 20px;
  align-items: center;
  margin-top: 20px;
  .chat-container {
    height: 300px;
    width: 600px;
    border: 1px solid black;
  }
  .el-alert {
    width: 400px;
  }
}
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
