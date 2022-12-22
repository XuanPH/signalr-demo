<script setup lang="ts">
import {
  HubConnectionBuilder,
  LogLevel,
  HttpTransportType,
  HubConnection,
} from "@microsoft/signalr";
import { computed, reactive, ref } from "vue";
import HelloWorld from "./components/HelloWorld.vue";

const roomId = "9520ec72-8faf-4157-886d-4b82490c18c9";
const userAppId = "3368637342326461234";
const userName = "User Name";
const userAvatar = "https://h5.zdn.vn/static/images/avatar.png";

const hub = ref<HubConnection>({} as HubConnection);
const payload = ref<any>();
const facing = ref<number>(0.5);
let pureUrl = `${import.meta.env.VITE_ENDPOINT}/hubs/game?roomId=${roomId}&userAppId=${userAppId}&userName=${userName}&userAvatar=${userAvatar}`;
hub.value = new HubConnectionBuilder()
  .withUrl(pureUrl, {
    skipNegotiation: true,
    transport: HttpTransportType.WebSockets,
  })
  .configureLogging(LogLevel.Information)
  .build();

const payloadStringify = computed(() => {
  JSON.stringify(payload.value || {});
});

// await _this.start();
// retry
hub.value.onclose(async () => {
  await start();
});

const testInvokeMove = async () => {
  await hub.value.invoke("move", facing.value, roomId);
};
const testInvokeRoom = async () => {
  await hub.value.invoke("invoke-room", roomId);
};

hub.value.on("move", (payload: any) => {
  console.log("move", payload);
  payload.value = {
    method: "move",
    ...payload,
  };
});
hub.value.on("room", (payload: any) => {
  console.log("room", payload);
  payload.value = {
    method: "room",
    ...payload,
  };
});

const start = async () => {
  try {
    console.log("Attempting reconnect");
    await hub.value.start();
  } catch (err) {
    console.log(err);
    setTimeout(async () => {
      await start();
    }, 5000);
  }
};
</script>

<template>
  <div style="display: flex; flex-direction: column">
    <button
      class="btn"
      @click.prevent="start"
      :disabled="hub.state !== 'Disconnected'"
    >
      Connect Hub
    </button>
    <button
      :disabled="hub.state !== 'Connected'"
      @click.prevent="testInvokeMove()"
    >
      Invoke Move
    </button>
    <button
      :disabled="hub.state !== 'Connected'"
      @click.prevent="testInvokeRoom()"
    >
      Invoke room
    </button>
  </div>

  <div class="state">
    <div>
      Trạng thái kết nối
      <span :class="hub.state.toLocaleLowerCase()">{{ hub.state }}</span>
    </div>
  </div>
</template>

<style scoped>
.state {
  margin-top: 30px;
}
button {
  margin-top: 15px;
}
.state > div {
  margin-bottom: 20px;
  font-size: 20px;
}
.state > div > span {
  font-weight: bold;
}
.state > div > span.disconnected {
  color: red;
}
.state > div > span.connecting {
  color: yellow;
}
.state > div > span.connected {
  color: yellowgreen;
}
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
