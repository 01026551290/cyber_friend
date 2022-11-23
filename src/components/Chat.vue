<template>
  <div class="chat_wrap">
    <div class="line" v-for="chat in chatList" :key="chat">
      <span class="chat-box" :class="[chat.mine ? 'mine' : '']">{{chat.text}}</span>
    </div>
  </div>
  <div class="chat_send_wrap">
    <input class="chat-box" v-model="chatText" @keyup.enter="sendChat">
    <button @click="sendChat" >말하기</button>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: "Chat",
  setup() {
    const chatHistory = ''
    const chatText = ''
    const chatList = []
    return {
      chatText,
      chatList,
      chatHistory
    };
  },
  watch: {
    chatList: () => {
      this.$forceUpdate();
    }
  },
  methods: {
    async sendChat() {
      this.chatList.push({
        text: this.chatText,
        mine: true
      })

      this.chatHistory === '' ? this.chatHistory += this.chatText :  this.chatHistory += '\n\n' + this.chatText

      await axios.post('https://api.openai.com/v1/completions', {
        model: "text-davinci-002",
        prompt: this.chatHistory,
        temperature: 0.7,
        max_tokens: 256,
        top_p: 1,
        frequency_penalty: 0,
        presence_penalty: 0,
      }, {
        headers: {
          Authorization: 'Bearer ' + process.env.VUE_APP_CLIENT_SECRET
        }
      }).then(response => {
        this.chatList.push({
          text: response.data.choices[0].text,
          mine: false
        })
      })

      this.chatText = ''
      await this.$forceUpdate();
    },
  }
}
</script>

<style scoped>
.chat_wrap {
  width: 400px;
  background-color: skyblue;
  height: 100vh;
  padding-top: 2px;
  overflow: auto;
}
.line {
  display: flex;
  margin: 15px 10px 0px 10px;
}
.chat-box { background: #eee;
  padding: 5px;
  max-width: 200px;
}
.mine { margin-left: auto }
.chat_send_wrap {
  width: 400px;
}
</style>