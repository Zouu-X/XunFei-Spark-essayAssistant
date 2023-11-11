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
import {getWebsocketUrl} from "@/views/mainPage/api";
import {requestObj} from "@/views/mainPage/api";
export default {
  name: "index",
  data() {
    return {
      messages: [],
      showClosingAlert: false,
      textInput: '',
      radio: 'TEST1',
      localRequestObj: requestObj
    }
  },

  methods: {
    open() {
      this.$alert("Zoe", "联系方式", {confirmButtonText: '确定'})
    },
    //面谱选择逻辑
    inputListener() {
      console.log(this.radio)
    },
    Submit() {
      if (this.textInput === '') {
        this.showClosingAlert = true
        setTimeout(() => {
          this.showClosingAlert = false
        }, 2000)
      }else {
        this.messages.push({ text: this.textInput, isMine: true })
        this.sendMsg(this.textInput)
        this.textInput = ''
      }
    },
    async sendMsg (textInput) {
      let myUrl = await getWebsocketUrl();
      let socket = new WebSocket(myUrl);
      let tempMsg = ''
      socket.addEventListener('open', (event) => {
        let params = {
          "header": {
            "app_id": this.localRequestObj.APPID,
            "uid": "Vycery"
          },
          "parameter": {
            "chat": {
              "domain": "general",
              "temperature": 0.5,
              "max_tokens": 1024,
            }
          },
          "payload": {
            "message": {
              // 如果想获取结合上下文的回答，需要开发者每次将历史问答信息一起传给服务端，如下示例
              // 注意：text里面的所有content内容加一起的tokens需要控制在8192以内，开发者如有较长对话需求，需要适当裁剪历史信息
              "text": [
                { "role": "user", "content": "你是谁" }, //# 用户的历史问题
                { "role": "assistant", "content": "我是AI助手" },  //# AI的历史回答结果
                // ....... 省略的历史对话
                { "role": "user", "content": textInput },  //# 最新的一条问题，如无需上下文，可只传最新一条问题
              ]
            }
          }
        };
        console.log("发送消息");
        socket.send(JSON.stringify(params))
      })
      socket.addEventListener('message', (event) => {
        let data = JSON.parse(event.data)
        console.log('收到消息！！',data);
        this.localRequestObj.sparkResult += data.payload.choices.text[0].content

        tempMsg += data.payload.choices.text[0].content
        if (data.header.code !== 0) {
          console.log("出错了", data.header.code, ":", data.header.message);
          // 出错了"手动关闭连接"
          socket.close()
        }
        if (data.header.code === 0) {
          // 对话已经完成
          if (data.payload.choices.text && data.header.status === 2) {
            // this.localRequestObj.sparkResult += data.payload.choices.text[0].content;
            setTimeout(() => {
              // "对话完成，手动关闭连接"
              socket.close()
            }, 1000)
          }
        }
        // this.messages.push({ text: this.localRequestObj.sparkResult, isMine: false })
        // console.log('MSGRES', this.messages)
        // addMsgToTextarea(localRequestObj.sparkResult);
      })
      socket.addEventListener('close', (event) => {
        console.log('连接关闭！！', event);
        console.log('sparkRes', this.localRequestObj.sparkResult)
        this.messages.push({ text: tempMsg, isMine: false })
        // 对话完成后socket会关闭，将聊天记录换行处理
        // this.localRequestObj.sparkResult = this.localRequestObj.sparkResult + "&#10;"
        // addMsgToTextarea(localRequestObj.sparkResult);
      })
      socket.addEventListener('error', (event) => {
        console.log('连接发送错误！！', event);
      })
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
  justify-content: space-between;
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
