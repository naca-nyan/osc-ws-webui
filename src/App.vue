<script setup lang="ts">
import { ref } from "vue";

interface ConnectionState {
  state: "not connected" | "connecting" | "connected";
}

const state = ref<ConnectionState>({ state: "not connected" });

const serverAddr = ref("ws://");
const parameter = ref("/avatar/parameters/hoge");
const value = ref("1");

let _sock: WebSocket;

function connect() {
  const addr = serverAddr.value;
  const sock = new WebSocket(addr);
  _sock = sock;
  sock.addEventListener("open", () => {
    state.value.state = "connected";
  });
  sock.addEventListener("message", async (message) => {
    const data = await JSON.parse(message.data);
    const { path, value } = data;
    fetch("http://localhost:9090/", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        path: path,
        value: value,
      }),
    });
  });
}

function send() {
  _sock.send(
    JSON.stringify({
      path: parameter,
      value: value,
    })
  );
}
</script>

<template>
  <div v-if="state.state === 'connected'">
    <label for="parameter">parameter</label>
    <input v-model="parameter" />
    <label for="value">value</label>
    <input v-model="value" />
    <button @click="send">send</button>
  </div>
  <div v-else>
    <input type="text" v-model="serverAddr" />
    <button @click="connect">connect</button>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
