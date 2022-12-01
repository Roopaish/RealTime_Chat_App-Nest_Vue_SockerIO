<script setup lang="ts">
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";
import type { Message } from "./type";

// const socket = io("http://localhost:3001");
const socket = io("https://realtimechatapp-production.up.railway.app/");

const messages = ref<Message[]>([]);
const messageText = ref<string>("");
const joined = ref<boolean>(false);
const name = ref<string>("");
const typingDisplay = ref<string>("");

onBeforeMount(() => {
  socket.emit("findAllMessages", {}, (response: Message[]) => {
    messages.value = response;
  });

  socket.on("message", (message: Message) => {
    messages.value.push(message);
  });

  socket.on("typing", ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing...`;
    } else {
      typingDisplay.value = "";
    }
  });
});

const join = () => {
  socket.emit("join", { name: name.value }, () => {
    joined.value = true;
  });
};

const sendMessage = () => {
  socket.emit("createMessage", { text: messageText.value }, () => {
    messageText.value = "";
  });
};

let timeout;
const emitTyping = () => {
  socket.emit("typing", { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit("typing", { isTyping: false });
  }, 2000);
};
</script>
<template>
  <head>
    <link
      href="https://bouncecss.bookie0.repl.co/bounce.css"
      rel="stylesheet"
      type="text/css"
    />
  </head>
  <div class="chat">
    <h4 v-if="!joined" class="welcome">
      Welcome to real time chat app made with Nest and Vue made by
      <a href="https://github.com/roopaish" target="_blank">@Roopaish</a>
    </h4>
    <div v-if="!joined">
      <form @submit.prevent="join" class="form">
        <label><h2>What's your name?</h2></label>
        <input
          v-model="name"
          required
          type="text"
          placeholder="Enter your name..."
        />
        <button type="submit">Submit</button>
      </form>
    </div>
    <div class="chat-container" v-else>
      <div class="messages-container">
        <div v-for="message in messages">
          <div class="message-container">
            <h4 class="sender">[{{ message.name }}]:</h4>
            <p class="message">{{ message.text }}</p>
          </div>
        </div>
      </div>
      <h6 v-if="typingDisplay">
        {{ typingDisplay }}
      </h6>
      <div class="message-div">
        <form @submit.prevent="sendMessage" class="message-form">
          <input
            v-model="messageText"
            @input="emitTyping"
            required
            type="text"
            placeholder="Type your message here..."
          />
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style>
.welcome {
  margin-bottom: 20px;
}

.chat {
  padding: 20px;
  width: 100%;
  min-height: 100vh;
  display: grid;
  place-content: center;
}

.sender {
  display: inline;
}

.message {
  font-size: 24px;
  color: rgb(65, 65, 65);
  display: inline;
  margin-left: 10px;
  white-space: pre-wrap;
  /* font-family: Arial, Helvetica, sans-serif; */
}

.message-div {
  width: 100%;
}

.message-form {
  width: 100%;
  display: flex;
}

.message-form input {
  width: 100% !important;
  flex-grow: 1;
}

.chat-container {
  width: 100vw;
  max-width: 600px;
  margin: -20px;
  padding: 20px;
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.messages-container {
  flex: 1;
  overflow-y: scroll;
}

h1,
h2,
h3,
h4,
h5,
h6 {
  line-height: 1.3 !important;
}
</style>
