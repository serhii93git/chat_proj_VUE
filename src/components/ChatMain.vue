<template>
  <div class="chat-container">
    <div class="header">
      <h1>Chat Application</h1>
    </div>
    <div class="messages" ref="messagesContainer">
      <div v-for="(message, index) in messages" :key="index" class="message" :class="{'my-message': message.username === username, 'other-message': message.username !== username}">
        <div class="message-content">
          <strong>{{ message.username }}:</strong>
          <p>{{ message.content }}</p>
        </div>
      </div>
    </div>
    <div class="input-container">
      <input v-model="username" placeholder="Enter your username" class="input username-input" />
      <button @click="connectWebSocket" :disabled="isConnected" class="btn connect-btn">Connect</button>
      <input v-model="newMessage" @keyup.enter="sendMessage" placeholder="Type a message" :disabled="!isConnected" class="input message-input" />
      <button @click="sendMessage" :disabled="!isConnected" class="btn send-btn">Send</button>
    </div>
  </div>
  <button @click="toggleTheme" class="btn theme-toggle-btn">Toggle Theme</button>
</template>

<script>
import '../style/ChatMain.css'

export default {
  data() {
    return {
      websocket: null,
      username: "",
      newMessage: "",
      messages: [],
      isConnected: false
    };
  },
  methods: {



    toggleTheme() {
      document.body.classList.toggle('dark-theme');
    },


    connectWebSocket() {
      if (this.username.trim() === "") {
        alert("Please enter a username.");
        return;
      }
      const wsUrl = `ws://localhost:8000/ws/chat?username=${this.username}`;
      this.websocket = new WebSocket(wsUrl);

      this.websocket.onmessage = (event) => {
        const message = JSON.parse(event.data);
        this.messages.push(message);
        this.$nextTick(() => {
          // Автоматичне перемотування на останнє повідомлення
          const container = this.$refs.messagesContainer;
          container.scrollTop = container.scrollHeight;
        });
      };

      this.websocket.onclose = () => {
        console.log("WebSocket connection closed");
        this.isConnected = false;
      };

      this.websocket.onopen = () => {
        console.log("WebSocket connection opened");
        this.isConnected = true;
      };
    },
    sendMessage() {
      if (this.websocket && this.newMessage.trim() !== "") {
        this.websocket.send(this.newMessage);
        this.newMessage = "";
      }
    }
  }
};
</script>