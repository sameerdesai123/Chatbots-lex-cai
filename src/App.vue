<template>
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
</template>

<script>
import VueBotUI from './components/BotUI.vue'

export default {
  name: 'ChatBot',
  data () {
    return {
      sendMsg: '',
      messageData: [],
      messageService: null,
      botTyping: false,
      lexUserId: '',
      btnMapper: null,
      sessionAttributes: {},
      inputDisable: false,
      botOptions: {
        botTitle: 'AMS Bot',
        colorScheme: '#FFA500',
        boardContentBg: '#f4f4f4',
        msgBubbleBgBot: '#fff',
        inputPlaceholder: 'Type here',
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
        text: 'Starter text'
      })
    },
    deleteSession () {
      if (this.messageData.length === 1) {
        return
      }
      var params = {
        botAlias: '#### some-alias ####' /* required */,
        botName: '#### your-bot-name ####' /* required */,
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
      this.sessionAttributes = { ORIGIN: 'WEB'}
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
      console.log(this.btnMapper != null);
      if(this.btnMapper != null) this.sendMsg = this.btnMapper[value.text];
      this.getResponse()
      this.sendMsg = ''
      this.mapper(null);
    },
    mapper(btns) {
        if(btns == null) this.btnMapper = null;
        else{
            let map = {};
            btns.forEach(btn => {
                map[btn.text] = btn.value;
            })
            this.btnMapper = map;
        }
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

      var replyMessage = [];
      var params = {
        botAlias: '#### some-alias ####' /* required */,
        botName: '#### your-bot-name ####' /* required */,
        inputText: this.sendMsg,
        userId: this.lexUserId,
        sessionAttributes: this.sessionAttributes
      }
        console.log(params);
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
            replyMessage.push({
              agent: 'bot',
              type: 'text',
              disableInput: false,
              text: data.message
            });
          } else if (data.responseCard.genericAttachments[0].attachmentLinkUrl !== null) {
            console.log("Entered block for link");
            
            replyMessage.push({
              agent: 'bot',
              type: 'text',
              text: data.message,
              disableInput: false,
            });
            console.log("Entered block for link");
            replyMessage.push({
              agent: 'bot',
              type: 'link',
              text: data.responseCard.genericAttachments[0].attachmentLinkUrl,
              disableInput: false,
            });
          }
          else {
            console.log('Response Card :', data.responseCard.genericAttachments[0])
            this.mapper(data.responseCard.genericAttachments[0].buttons);
            replyMessage.push({
              agent: 'bot',
              type: 'button',
              text: data.message,
              disableInput: true,
              options: this.getDisplayOptions(data.responseCard.genericAttachments[0].buttons)
            });
          }
          console.log('Pushing Message Now', replyMessage);
          // this.messageData.push(replyMessage)
          replyMessage.forEach(reply => {
            this.messageData.push(reply);
          });
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

<style scoped>
</style>
