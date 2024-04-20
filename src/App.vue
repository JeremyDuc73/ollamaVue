<template>
  <div class="chat-container">
    <div class="messages" ref="messages">
      <div v-for="(message, index) in chatMessages" :key="index" :class="message.from === 'user' ? 'user-message' : 'ollama-message'">
        {{ message.text }}
      </div>
    </div>
    <input v-model="userInput" @keyup.enter="sendMessage" placeholder="Type your message..." />
    <button @click="sendMessage">Send</button>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      chatMessages: [],
      userInput: ''
    };
  },
  methods: {
    async sendMessage() {
      const userMessage = this.userInput.trim();
      if (userMessage === '') return;

      // Préparez les données à envoyer dans la requête POST
      const requestData = {
        model: 'mistral',
        prompt: userMessage,
        stream: false
      };

      try {
        const response = await axios.post('http://localhost:11434/api/generate', requestData);
        const ollamaResponse = response.data.response;
        this.chatMessages.push({text: userMessage, from: 'user'});
        this.chatMessages.push({text: ollamaResponse, from: 'ollama'});
        this.userInput = '';

        this.$nextTick(() => {
          this.$refs.messages.scrollTop = this.$refs.messages.scrollHeight;
        });
      } catch (error) {
        console.error('Error while calling Mistral API:', error);
      }
    }
  }
};
</script>

<style >
.chat-container {
  width: 800px;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
}

.messages {
  max-height: 300px;
  overflow-y: auto;
}

.user-message {
  text-align: right;
  margin-bottom: 10px;
}

.ollama-message {
  text-align: left;
  margin-bottom: 10px;
}
</style>
