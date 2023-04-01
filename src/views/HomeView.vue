<template>
  <div class="container">
    <div class="chat-list">
      <div class="chat-item" v-for="item in chatList" :key="item.topic" @click="clickChatListItem(item.id)">
        <div class="chat-item-header">
          <div class="chat-item-header-title">{{ item.topic }}</div>
        </div>
      </div>
    </div>
    <div class="chat-main">
      <div class="chat-detail">
        <div class="chat-item-header">
          <div class="chat-item-header-title">{{ chatList[current_index].topic }}</div>
          <div class="chat-item-header-time">{{ chatList[current_index].chat_time }}</div>
        </div>
        <div class="chat-item-body">
          <div class="chat-item-body-message" v-for="message in chatList[current_index].message" :key="message.msg_id">
            <div class="chat-item-body-message-content" :class="message.role">{{ message.content }}</div>
            <div class="chat-item-body-message-time" :class="message.role">{{ message.time }}</div>
          </div>
        </div>
      </div>
      <div class="chat-input">
        <input type="text" placeholder="请输入内容" v-model="msg">
        <button @click="sendMsg">发送</button>
      </div>
    </div>

  </div>
</template>

<script>
export default {
  name: 'HomeView',
  data() {
    return {
      current_index: 0,
      msg: '',
      chatList: [
        {
          "topic": "新的主题",
          "id": "1",
          "chat_time": "2021-01-01 12:00:00",
          "message": [
            {"role": "user", "content": "你是谁", "time": "2021-01-01 12:00:00", "msg_id": "78f03745-3562-4f69-b053-06d8beb899de"},
            {"role": "assistant", "content": "我是一个AI机器人，我被设计用来回答各种问题，帮助人们解决各种问题。", "time": "2021-01-01 12:00:00"},
            {"role": "user", "content": "你是谁", "time": "2021-01-01 12:00:00", "msg_id": "78f03745-3562-4f69-b053-06d8beb899de"},
            {"role": "assistant", "content": "我是一个AI机器人，我被设计用来回答各种问题，帮助人们解决各种问题。", "time": "2021-01-01 12:00:00"},

          ]
        },
        {
          "topic": "新的主题2",
          "id": "2",
          "chat_time": "2021-01-01 12:00:00",
          "message": [
            {"role": "user", "content": "你是谁", "time": "2021-01-01 12:00:00", "msg_id": "78f03745-3562-4f69-b053-06d8beb899de"},
            {"role": "assistant", "content": "我是一个AI机器人，我被设计用来回答各种问题，帮助人们解决各种问题。", "time": "2021-01-01 12:00:00"},
          ]
        }
      ]
    }
  },
  methods: {
    clickChatListItem(id) {
      this.current_index = this.chatList.findIndex(item => item.id === id)
    },
    sendMsg() {
      this.chatList[this.current_index].message.push({
        "role": "user",
        "content": this.msg,
        "time": this.getNowTime(),
        "msg_id": "78f03745-3562-4f69-b053-06d8beb899de"
      })
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
    }
  }
}
</script>

<style lang="less">
.container {
  display: flex;
  .chat-list {
    display: flex;
    flex-direction: column;
    //background-color: #f4f4f4;
    height: 500px;

    .chat-item {
      background-color: rgba(81, 203, 213, 0.66);
      color: white;
      border-radius: 5px;
      padding: 10px;
      margin: 2px;
      width: 7rem;
      overflow: hidden;
      cursor: pointer;
    }
  }
  .chat-main {
    flex: 1;
    display: flex;
    flex-direction: column;
    .chat-detail {
      flex: 1;
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
      }

      .chat-item-body {
        height: 500px;
        overflow-y: auto;
        padding: 10px;
        .chat-item-body-message {
          display: flex;
          flex-direction: column;
          margin-bottom: 10px;
          .chat-item-body-message-content {
            padding: 10px;
            border-radius: 5px;
            &.user {
              background-color: #f4f4f4;
              color: #333;
              align-self: flex-end;
            }
            &.assistant {
              background-color: #00a2ff;
              color: white;
              align-self: flex-start;
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
      input {
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

