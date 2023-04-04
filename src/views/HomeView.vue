<template>
  <div class="container">
    <div class="left">
      <div class="add-topic">
        <button @click="addTopic">添加主题</button>
      </div>
      <div class="chat-list">
        <div class="chat-item" :class="{activate: index == current_index}" v-for="(item, index) in chatList" :key="item.id" @click="clickChatListItem(item.id)">
          <div class="chat-item-header">
            <div class="chat-item-header-title">{{ item.topic }}</div>
          </div>
        </div>
      </div>
    </div>

    <div class="chat-main">
      <div class="chat-item-header">
        <div class="chat-item-header-time">{{ chatList[current_index].chat_time }}</div>
        <div class="chat-item-header-title">{{ chatList[current_index].topic }}</div>
        <div class="delete" @click="deleteTopic(chatList[current_index].id)">删除主题</div>
      </div>
      <div class="chat-detail" ref="chatBodyRef">
        <div class="chat-item-body">
          <div class="chat-item-body-message" v-for="message in chatList[current_index].message" :key="message.msg_id">
            <div class="chat-item-body-message-content" :class="message.role" v-html="markdown(message.content)"></div>
            <div class="chat-item-body-message-time" :class="message.role">{{ message.time }}</div>
          </div>
        </div>
      </div>
      <div class="chat-input">
        <textarea @keydown.enter="keyDown" :disabled="chatList[current_index].sending" placeholder="请输入内容 按住Enter键发送消息,按住Ctrl+Enter键换行" v-model="msg" />
        <button @click="sendMsg(chatList[current_index].id)">发送</button>
      </div>
      <p style="color:#b2afaf; fontSize:10px">测试中,所有的聊天记录可以在本地的localStorage中找到。可以同时开多个主题进行对话，随意切换</p>
    </div>

  </div>
</template>

<script>
import { marked } from 'marked'
import hljs from "highlight.js"; // 引入 highlight.js
import "highlight.js/styles/atom-one-dark.css"; // 引入高亮样式 这里我用的是sublime样式

// var rendererMD = new marked.Renderer();
// marked.setOptions({
//   renderer: rendererMD,
//   highlight: function(code) {
//     return hljs.highlightAuto(code).value;
//   },
//   pedantic: false,
//   gfm: true,
//   tables: true,
//   breaks: false,
//   sanitize: false,
//   smartLists: true,
//   smartypants: false,
//   xhtml: false
// });
var rendererMD = new marked.Renderer();
marked.setOptions({
  renderer: rendererMD,
  highlight: function(code) {
    return hljs.highlightAuto(code).value;
  },
  langPrefix: 'hljs language-', // highlight.js css expects a top-level 'hljs' class.
  tables: true,
  gfm: true,
  breaks: true,
})

import { nanoid } from 'nanoid'

export default {
  name: 'HomeView',
  data() {
    return {
      current_index: 0,
      msg: '',
      chatList: [
        {
          "topic": "新的主题",
          "id": nanoid(),
          "chat_time": this.getNowTime(),
          "sending": false,
          "message": [
            {"role": "assistant", "content": "你好，很高兴认识你，随便问点什么吧！", "time": this.getNowTime()},

          ]
        },
        // {
        //   "topic": "新的主题2",
        //   "id": "2",
        //   "chat_time": "2021-01-01 12:00:00",
        //   "message": [
        //     {"role": "user", "content": "你是谁", "time": "2021-01-01 12:00:00", "msg_id": "78f03745-3562-4f69-b053-06d8beb899de"},
        //     {"role": "assistant", "content": "我是一个AI机器人，我被设计用来回答各种问题，帮助人们解决各种问题。", "time": "2021-01-01 12:00:00"},
        //   ]
        // }
      ]
    }
  },
  watch: {
    current_index() {
      this.refreshLast()
      this.toBottom()
    }
  },
  mounted() {
    this.loadChatList()
    this.refreshLast()
  },
  methods: {
    refreshLast() {
      if(this.chatList[this.current_index].message[this.chatList[this.current_index].message.length - 1].role == 'assistant') {
        // 删除最后一条assistant的message
        this.chatList[this.current_index].message.pop()
      }
      // 如果chatList message的最后一条role不是assistant，就用msg_id 重新请求一下result
      if(this.chatList[this.current_index].message[this.chatList[this.current_index].message.length - 1].role != 'assistant') {
        let msg_id = this.chatList[this.current_index].message[this.chatList[this.current_index].message.length - 1].msg_id
        this.chatList[this.current_index].message.push({
          "role": "assistant",
          "content": '连接服务器中',
          "time": this.getNowTime()
        })
        this.getMsg(msg_id, this.chatList[this.current_index].id)
      }
    },
    deleteTopic(chat_id) {
      let index = this.getChatIndex(chat_id)
      this.chatList.splice(index, 1)
      this.saveChatList()
    },
    saveChatList() {
      localStorage.setItem('chatList', JSON.stringify(this.chatList))
    },
    loadChatList() {
      let chatList = localStorage.getItem('chatList')
      if(chatList) {
        this.chatList = JSON.parse(chatList)
      }
    },
    addTopic() {
      var topic = prompt("请输入主题", "新的主题");
      let nowTime = this.getNowTime()
      if (topic != null) {
        const id = nanoid() // => "V1StGXR8_Z5jdHi6B-myT"
        let newChat = {
          "topic": topic,
          "id": id,
          "chat_time": nowTime,
          "message": [
            {"role": "assistant", "content": "随便问点什么吧！", "time": nowTime},
          ]
        }
        if(topic !== '新的主题') {
          newChat["message"][0].content = `问点和${topic}相关的吧!`
        }

        this.chatList.unshift(newChat)
        this.saveChatList()
      }


    },
    markdown(content) {
      return marked(content)
    },
    keyDown(e) {
      if(e.ctrlKey && e.keyCode==13) {  //用户点击了ctrl+enter触发
        this.msg += '\n';
      }else { //用户点击了enter触发
        // 执行一些逻辑方法
        this.sendMsg(this.chatList[this.current_index].id)
        if(e != undefined){
          e.preventDefault();  // 阻止浏览器默认的敲击回车换行的方法
        }
      }
    },
    clickChatListItem(id) {
      this.current_index = this.getChatIndex(id)
    },
    getChatIndex(id) {
      return this.chatList.findIndex(item => item.id === id)
    },
    toBottom() {
      this.$nextTick(() => {
        this.$refs.chatBodyRef.scrollTop = this.$refs.chatBodyRef.scrollHeight
        console.log(this.$refs.chatBodyRef.scrollHeight)
      })
    },
    sendMsg(chat_id) {
      let current_index = this.getChatIndex(chat_id)
      this.chatList[current_index].sending = true
      this.chatList[current_index].message.push({
        "role": "user",
        "content": this.msg,
        "time": this.getNowTime(),
        "msg_id": ""
      })
      this.postSendMsg(this.msg, chat_id)
      this.msg = ''
      this.toBottom()
    },
    /**
     * 获取当前时间 时间格式"2021-01-01 12:00:00" 24小时制 有前导零
     * @returns {string}
     */
    getNowTime() {
      let date = new Date();
      let year = date.getFullYear();
      let month = date.getMonth() + 1;
      let day = date.getDate();
      let hour = date.getHours();
      let minute = date.getMinutes();
      let second = date.getSeconds();
      month = month < 10 ? "0" + month : month;
      day = day < 10 ? "0" + day : day;
      hour = hour < 10 ? "0" + hour : hour;
      minute = minute < 10 ? "0" + minute : minute;
      second = second < 10 ? "0" + second : second;
      return year + "-" + month + "-" + day + " " + hour + ":" + minute + ":" + second;
    },
    getMsg(msg_id, chat_id) {
      let current_index = this.getChatIndex(chat_id)


      fetch(`msg-api/result/${msg_id}`, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json'
        }
      }).then(res => {
        return res.json()
      }).then(res => {
        if (res.status === 'done') {
          this.chatList[current_index].message[this.chatList[current_index].message.length - 1].content = res.result
          this.chatList[current_index].sending = false
          this.toBottom()
        } else {
          if (res.status === 'pending') {
            // if(res.wait_tasks <= 1) {
            //   this.chatList[current_index].message[this.chatList[current_index].message.length - 1].content = `正在回答中`
            // } else {
            this.chatList[current_index].message[this.chatList[current_index].message.length - 1].content = `排队中，还剩${res.wait_tasks}个`
            // }
          } else if(res.status == 'processing') {
            this.chatList[current_index].message[this.chatList[current_index].message.length - 1].content = `正在回答中...`
          }
          setTimeout(() => {
            this.getMsg(msg_id, chat_id)
          }, 1000)
        }
      })
    },
    postSendMsg(msg, chat_id) {
      let current_index = this.getChatIndex(chat_id)

      let pre_id = ""
      if(this.chatList[current_index].message.length > 1) {
        for (let message of this.chatList[current_index].message) {
          console.log(message)
          if (message.role === 'user' && message.msg_id !== "") {
            pre_id = message.msg_id
            console.log(pre_id)
          }
        }
      }
      // 这里的pre_id 为什么为空 ？
      //

      console.log("pre_id", pre_id)

      // 用fatch发送消息post请求
      fetch('msg-api/message', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          "message": msg,
          "key": "88888888-88888888-88888888-88888888",
          "pre_id": pre_id,
        })
      }).then(res => {
        return res.json()
      }).then(res => {
        this.chatList[current_index].message[this.chatList[current_index].message.length - 1].msg_id = res.msg_id
        this.saveChatList()
        this.chatList[current_index].message.push({
          "role": "assistant",
          "content": '连接服务器中',
          "time": this.getNowTime()
        })
        this.getMsg(res.msg_id, chat_id)
        this.toBottom()
      })
    }
  }
}
</script>

<style lang="less">
.container {
  display: flex;
  height: 100%;
  .left {
    .add-topic {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 30px;
      background-color: #f4f4f4;
      cursor: pointer;
      button {
        border: none;
        background-color: #f4f4f4;
        font-size: 1rem;
        color: #333;
        cursor: pointer;
        user-select: none;
      }
      margin-bottom: 20px;
    }
    .chat-list {
      display: flex;
      flex-direction: column;
      //background-color: #f4f4f4;
      height: 500px;

      .chat-item {
        user-select: none;
        background-color: rgba(81, 203, 213, 0.66);
        color: white;
        border-radius: 5px;
        padding: 10px;
        margin: 2px;
        width: 7rem;
        overflow: hidden;
        cursor: pointer;
        &.activate {
          background-color: rgba(81, 147, 213, 0.66);
          color: white;
        }
      }
    }
  }
  .chat-main {
    flex: 1;
    display: flex;
    flex-direction: column;
    padding: 0 10%;
    width: 100%;


    .chat-item-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 10px;
      width: 100%;

      .chat-item-header-title {
        font-size: 1.2rem;
        font-weight: 500;
        margin-bottom: 0.4rem;
        color: var(--color-heading);
      }
      .chat-item-header-time {
        font-size: 0.8rem;
        color: #999;
      }
      .delete {
        cursor: pointer;
      }
    }

    .chat-detail {
      flex: 1;
      height: 100vh;
      overflow: auto;
      width: 100%;

      .chat-item-body {
        //height: 500px;
        overflow: auto;



        .chat-item-body-message {
          display: flex;
          flex-direction: column;
          margin-bottom: 10px;


          .chat-item-body-message-content {
            padding: 10px;
            border-radius: 5px;
            width: 100%;
            &.user {
              //background-color: #f4f4f4;
              //color: #333;
              background-color: #95ec69;
              color: #24243a;
              align-self: flex-end;
            }
            &.assistant {
              //background-color: #00a2ff;
              //color: white;
              background-color: #f4f4f4;
              color: #24243a;
              align-self: flex-start;
            }
            code {
              //background-color: #282c34;
              padding: 0 10px;
              border-radius: 5px;
            }
            ol, ul, dl {
              //list-style: none;
              //margin: 0;
              padding-left: 0;
              list-style-position: inside;
            }

          }
          .chat-item-body-message-time {
            font-size: 0.8rem;
            color: #999;
            &.user {
              align-self: flex-end;
            }
            &.assistant {
              align-self: flex-start;
            }
          }
        }
      }
    }
    .chat-input {
      display: flex;
      align-items: center;
      padding: 10px;
      textarea {
        flex: 1;
        border: 1px solid #ccc;
        border-radius: 5px;
        padding: 5px;
      }
      button {
        margin-left: 10px;
        padding: 5px 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
        background-color: #fff;
        cursor: pointer;
      }
    }

  }

}
</style>

