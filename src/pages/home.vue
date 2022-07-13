<template>
<f7-page>
  <f7-navbar title="Your Companion Voice Chat" />

  <f7-messages>
    <f7-messages-title>Press the Alan icon and start talking</f7-messages-title>
    <f7-message
      v-for="(message, index) in messagesData"
      :key="index"
      :type="message.type"
      :name="message.name"
      :avatar="message.avatar"
      :first="isFirstMessage(message, index)"
      :last="isLastMessage(message, index)"
      :tail="isTailMessage(message, index)"
    >
      <template #text>
        <!-- eslint-disable-next-line -->
        <span v-if="message.text" v-html="message.text"></span>
      </template>
    </f7-message>
  </f7-messages>
</f7-page>
</template>
<script>
import { f7ready } from 'framework7-vue';

export default {
  data() {
    return {
      messagesData: [],
      people: {
          name: 'Alan',
          avatar: 'https://avatars.githubusercontent.com/u/54960780?s=200&v=4',
      },
    };
  },
  mounted() {
    f7ready(() => {
      const self = this;
      document.addEventListener('deviceready', self.deviceReady, false);
    });
  },
  methods: {
    deviceReady() {
      const self = this;
      // Please input your ALAN API KEY for better customization
      const API_KEY = '1ee6a74d9cccea77faceb579449bc5142e956eca572e1d8b807a3e2338fdd0dc/stage'; // This is the free API Key provided by ALAN
      cordova.exec(
          self.alanData,
          self.alanDataError,
          'alanVoice',
          'addButton',
          [API_KEY]
      );
    },
    alanData(response) {
      const self = this;
      if (
        response.type === 'onEvent' &&
        response.data && 
        response.data.text
      ) {
        if (response.data.final && response.data.name === 'recognized') {
          self.sendMessage(response.data.text);
        } else if (response.data.ctx && response.data.ctx.success) {
          self.receiveMessage(response.data.text);
        }
      }
    },
    alanDataError(error) {
      alert('There are some error. Please try again');
      console.log(error);
    },
    isFirstMessage(message, index) {
      const self = this;
      const previousMessage = self.messagesData[index - 1];
      if (message.isTitle) return false;
      if (
        !previousMessage ||
        previousMessage.type !== message.type ||
        previousMessage.name !== message.name
      )
        return true;
      return false;
    },
    isLastMessage(message, index) {
      const self = this;
      const nextMessage = self.messagesData[index + 1];
      if (message.isTitle) return false;
      if (!nextMessage || nextMessage.type !== message.type || nextMessage.name !== message.name)
        return true;
      return false;
    },
    isTailMessage(message, index) {
      const self = this;
      const nextMessage = self.messagesData[index + 1];
      if (message.isTitle) return false;
      if (!nextMessage || nextMessage.type !== message.type || nextMessage.name !== message.name)
        return true;
      return false;
    },
    sendMessage(text) {
      const self = this;
      const messagesToSend = [];
      messagesToSend.push({
        text,
      });
      self.messagesData.push(...messagesToSend);
    },
    receiveMessage(answer) {
      const self = this;
      const person = self.people;
      self.messagesData.push({
        text: answer,
        type: 'received',
        name: person.name,
        avatar: person.avatar,
      });
    }
  },
};
</script>