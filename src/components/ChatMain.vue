<template>
  <div class="chat-container">
    <div class="header">
      <h1>Chat Application</h1>
    </div>
    <div class="messages" ref="messagesContainer">
      <!-- Перевіряємо наявність повідомлень та показуємо або історію повідомлень, або повідомлення про відсутність повідомлень -->
      <div v-if="messages.length === 0 && isConnected" class="no-messages">
        No messages yet. Be the first to send a message!
      </div>
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
    async fetchMessageHistory() {
      try {
        const response = await fetch(`http://localhost:8000/messages/${this.username}`, {
          headers: {
            'Content-Type': 'application/json'
          }
        });
        if (response.ok) {
          const history = await response.json();
          this.messages = history.map(msg => ({
            username: msg.username,  
            content: msg.content,
            send_time: msg.send_time
          }));
          this.$nextTick(() => {
            // Автоматичне перемотування на останнє повідомлення
            const container = this.$refs.messagesContainer;
            container.scrollTop = container.scrollHeight;
          });
        } else {
          console.error('Failed to fetch message history. Status:', response.status);
          alert('Failed to fetch message history.');
        }
      } catch (error) {
        console.error('Failed to fetch message history:', error);
        alert('Failed to fetch message history.');
      }
    },
    async connectWebSocket() {
      if (this.username.trim() === "") {
        alert("Please enter a username.");
        return;
      }

      await this.fetchMessageHistory(); // Fetch message history before connecting to WebSocket

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
    },
    toggleTheme() {
      document.body.classList.toggle('dark-theme');
    }
  }
};
</script>

