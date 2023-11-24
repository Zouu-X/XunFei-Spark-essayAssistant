<template>
  <main>
    <div class="main-body">
      <div class="chat-container">
        <div class="chat-messages">
          <div
            v-for="(message, index) in messages"
            :key="index"
            class="msg-group"
            v-if="!message.invisible"
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
      <div class="mod-choose">
        <el-radio-group
            v-model="radio"
            @input="inputListener"
            style="margin-top: 30px;"
        >
          <el-radio-button label="全部文书"></el-radio-button>
          <el-radio-button label="简历"></el-radio-button>
          <el-radio-button label="动机信"></el-radio-button>
          <el-radio-button label="推荐信"></el-radio-button>
        </el-radio-group>
        <el-button class="mod-button" @click="modConfirm">确认</el-button>
      </div>

    </div>
  </main>

</template>

<script>
import {getWebsocketUrl} from "@/views/mainPage/api";
import {requestObj} from "@/views/mainPage/api";
export default {
  name: "index",
  props: ['goNewChat'],
  data() {
    return {
      dark: false,
      messages: [],
      showClosingAlert: false,
      textInput: '',
      radio: '',
      localRequestObj: requestObj,

    }
  },
  watch: {
    goNewChat(newVal) {
      if (newVal) {
        this.newChat()
      }
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
    modConfirm() {
      if(this.radio === '全部文书') {
        this.messages.push({role: "user", content: '我正在申请国外大学的硕士，我现在需要你成为一名专业的文书老师，教导我关于文书撰写的知识。请先向我了解具体要写什么文书，是简历、动机信还是推荐信或者其他的', invisible: true})
      }else if(this.radio === '简历') {
        this.messages.push({role: "user", content: '我正在做国外大学的硕士申请，我现在需要你成为一名专业的CV老师，教我写CV，首先告诉我CV要包含哪些内容，然后告诉我一步一步该怎么做', invisible: true})
      }else if(this.radio === '动机信') {
        this.messages.push({role: "user", content: '我正在做国外大学的硕士申请，我现在需要你成为一名专业的动机信老师，教我写动机信', invisible: true})
      }else if(this.radio === '推荐信') {
        this.messages.push({role: "user", content: '我正在做国外大学的硕士申请，我现在需要你成为一名专业的推荐信老师，一步一步教我拟一封来自导师的推荐信', invisible: true})
      }
      this.sendMsg()
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
              "temperature": 0.2,
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
.main-body {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
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

.mod-choose {
  display: flex;
  align-items: center;
  justify-content: center;
  .mod-button {
    margin-left: 6px;
    align-self: flex-end;
    background-color: lightblue;
  }
}
</style>
