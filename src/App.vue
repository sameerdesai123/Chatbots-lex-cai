<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link>|
      <router-link to="/about">About</router-link>
    </div>
    <router-view />
    <img src="./assets/logo.png" alt="Chat Icon" />
    <VueBotUI
      :options="botOptions"
      :messages="messageData"
      :bot-typing="botTyping"
      :input-disable="inputDisable"
      :is-open="false"
      @init="botStart"
      @msg-send="msgSend"
      @destroy="deleteSession"
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
      sendMsg: '',
      messageService: null,
      botTyping: false,
      lexUserId: '',
      sessionAttributes: {},
      inputDisable: false,
      botOptions: {
        botTitle: 'Order a Pizza',
        colorScheme: '#FFA500',
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
    deleteSession () {
      if (this.messageData.length === 1) {
        return
      }
      var params = {
        botAlias: 'PizzaLexBot' /* required */,
        botName: 'pizzabot' /* required */,
        userId: this.lexUserId /* required */
      }
      this.messageService.deleteSession(params, (err, data) => {
        if (err) console.log(err, err.stack)
        // an error occurred\
        else console.log('Deleted Session : ', data) // successful response
      })
      this.messageData = []
      this.lexUserId = ''
    },
    configService () {
      var AWS = require('aws-sdk')
      AWS.config.region = process.env.VUE_APP_REGION // Region
      AWS.config.credentials = new AWS.CognitoIdentityCredentials({
        // Provide your Pool Id here
        IdentityPoolId: process.env.VUE_APP_IdentityPoolId
      })

      var lexruntime = new AWS.LexRuntime()
      this.lexUserId = 'chatbot-demo' + Date.now()
      this.sessionAttributes = {}
      this.messageService = lexruntime // chatbot service API
    },

    msgSend (value) {
      // Push the user's message to board
      this.messageData.push({
        agent: 'user',
        type: 'text',
        text: value.text
      })
      this.sendMsg = value.text
      this.getResponse()
      this.sendMsg = ''
    },
    getDisplayOptions (options) {
      const opt = []
      options.forEach(element => {
        element.action = 'postback'
        opt.push(element)
      })
      return opt
    },

    // Submit the message from user to bot API, then get the response from Bot
    getResponse () {
      // Loading
      this.botTyping = true

      var replyMessage = {}
      var params = {
        botAlias: 'PizzaLexBot', // '$LATEST',
        botName: 'pizzabot',
        inputText: this.sendMsg,
        userId: this.lexUserId,
        sessionAttributes: this.sessionAttributes
      }

      // Post the message from user here
      this.messageService.postText(params, (err, data) => {
        if (err) {
          console.log(err, err.stack)
          console.log('Error:  ' + err.message + ' (see console for details)')
        }
        if (data) {
          // capture the sessionAttributes for the next cycle
          this.sessionAttributes = data.sessionAttributes
          // show response and/or error/dialog status
          console.log('Response : ', data)
          // Then get the response as below
          if (typeof data.responseCard === 'undefined') {
            replyMessage = {
              agent: 'bot',
              type: 'text',
              disableInput: false,
              text: data.message
            }
          } else {
            console.log('Response Card :', data.responseCard.genericAttachments[0])
            replyMessage = {
              agent: 'bot',
              type: 'button',
              text: data.message,
              disableInput: true,
              options: this.getDisplayOptions(data.responseCard.genericAttachments[0].buttons)
            }
          }
          console.log('Got Reply Now')
          console.log('Pushing Message Now')
          this.messageData.push(replyMessage)
          this.inputDisable = replyMessage.disableInput
          // finish
          this.botTyping = false
        }
      })
      // this.inputDisable = response.disableInput
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
