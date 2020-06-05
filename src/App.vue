<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div>
    <router-view/>
    <img src="./assets/logo.png" alt="Chat Icon">
    <VueBotUI
    :options="botOptions"
    :messages="messageData"
    :bot-typing="botTyping"
    :input-disable="inputDisable"
    :is-open="false"
    @init="botStart"
    @msg-send="msgSend"
    />
  </div>
</template>

<script>
import BotIcon from './assets/logo.png'
import { VueBotUI } from 'vue-bot-ui'

export default {
  data () {
    return {
      messageData: [],
      messageService: null,
      botTyping: false,
      inputDisable: false,
      botOptions: {
        botAvatarImg: BotIcon,
        boardContentBg: '#f4f4f4',
        msgBubbleBgBot: '#fff',
        inputPlaceholder: 'Type hereeee...',
        inputDisableBg: '#fff',
        inputDisablePlaceholder: 'Hit the buttons above to respond'
      }
    }
  },
  components: {
    VueBotUI
  },

  methods: {
    botStart () {
      this.botTyping = true
      // Complete Bot config and init
      this.configService()
      // Fake typing for the first message
      this.botTyping = false
      this.messageData.push({
        agent: 'bot',
        type: 'text',
        text: 'Hello'
      })
    },
    configService () {
      var AWS = require('aws-sdk')
      console.log(AWS)
      console.log(process.env.VUE_APP_SOMEKEY)
      this.messageService = {} // chatbot service API
    },

    msgSend (value) {
      // Push the user's message to board
      this.messageData.push({
        agent: 'user',
        type: 'text',
        text: value.text
      })

      this.getResponse()
    },

    // Submit the message from user to bot API, then get the response from Bot
    getResponse () {
      // Loading
      this.botTyping = true

      // Post the message from user here
      // Then get the response as below

      // Create new message from fake data
      this.messageService.createMessage().then(response => {
        const replyMessage = {
          agent: 'bot',
          ...response
        }

        this.inputDisable = response.disableInput
        this.messageData.push(replyMessage)

        // finish
        this.botTyping = false
      })
    }
  }
}
</script>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
