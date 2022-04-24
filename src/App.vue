<script setup lang="ts">
// eslint-disable-next-line @typescript-eslint/ban-ts-comment
//@ts-ignore
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";
const socket = io("http://localhost:3001");

const messages = ref([]);
const messagesText = ref("");
const joined = ref(false);
const name = ref("");
const typingDisplay = ref("");

onBeforeMount(() => {
  socket.emit("findAllMessages", {}, (response) => {
    messages.value = response;
  });

  socket.on("message", (message) => {
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
  socket.emit(
    "join",
    {
      name: name.value,
    },
    () => {
      joined.value = true;
    }
  );
};

const sendMessage = () => {
  socket.emit("createMessage", { text: messagesText.value }, () => {
    messagesText.value = "";
  });
};

let timeout: any;
const emitTyping = () => {
  socket.emit("typing", { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit("typing", { isTyping: false });
  }, 2000);
};
</script>

<template>
  <div class="chat">
    <div class="chat-container">
      <div class="messages-container">
        <div v-for="(message, key) in messages" :key="key" class="message">
          [{{ message.name }}]: {{ message.text }}
        </div>
      </div>
      <div class="typing-display">
        {{ typingDisplay }}
      </div>
      <div v-if="!joined">
        <form @submit.prevent="join">
          <label>Whats your name?</label>
          <input v-model="name" required />
          <button type="submit">Send</button>
        </form>
      </div>
      <div v-else>
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messagesText" @input="emitTyping" />
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style>
@import "@/assets/base.css";

.chat {
  padding: 20px;
  height: 100vh;
}
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}
.messages-container {
  flex: 1;
}

#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;

  font-weight: normal;
}

header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

a,
.green {
  text-decoration: none;
  color: hsla(160, 100%, 37%, 1);
  transition: 0.4s;
}

@media (hover: hover) {
  a:hover {
    background-color: hsla(160, 100%, 37%, 0.2);
  }
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  body {
    display: flex;
    place-items: center;
  }

  #app {
    display: grid;
    grid-template-columns: 1fr 1fr;
    padding: 0 2rem;
  }

  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
