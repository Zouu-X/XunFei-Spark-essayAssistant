<template>
  <main>
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
      <el-button class="style-change" @click="newChat">New Chat</el-button>
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
              <span v-if="message.role === 'user'">user</span>
              <span v-else-if="message.role === 'assistant'">AI</span>
            </div>
            <div
              class="message"
              :class="{ 'my-message' : message.role === 'user'}"
            >
              {{ message.content }}
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
  </main>

</template>

<script>
import {getWebsocketUrl} from "@/views/mainPage/api";
import {requestObj} from "@/views/mainPage/api";
export default {
  name: "index",
  data() {
    return {
      dialogVisible: false,
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
        this.$message({
          message: '请输入内容',
          type: 'warning'
        });
      }else {
        this.messages.push({role: "user", content: this.textInput})
        this.sendMsg()
        this.textInput = ''
      }
    },
    async sendMsg () {
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
              "domain": "generalv3",
              "temperature": 0.5,
              "max_tokens": 1024,
            }
          },
          "payload": {
            "message": {
              // 如果想获取结合上下文的回答，需要开发者每次将历史问答信息一起传给服务端，如下示例
              // 注意：text里面的所有content内容加一起的tokens需要控制在8192以内，开发者如有较长对话需求，需要适当裁剪历史信息
              "text": this.messages
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
      })
      socket.addEventListener('close', (event) => {
        console.log('连接关闭！！', event);
        console.log('sparkRes', this.localRequestObj.sparkResult)
        this.messages.push({role: 'assistant', content: tempMsg})
        // 对话完成后socket会关闭，将聊天记录换行处理
        // this.localRequestObj.sparkResult = this.localRequestObj.sparkResult + "&#10;"
        // addMsgToTextarea(localRequestObj.sparkResult);
      })
      socket.addEventListener('error', (event) => {
        console.log('连接发送错误！！', event);
      })
    },
    newChat() {
      this.messages = []
      this.localRequestObj.sparkResult = ''
      location.reload();
    }
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
.main-body {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
}
.contact-items {
  display: flex;
  row-gap: 4px;
  flex-direction: column;
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
