<template>
  <div class="flex flex-col h-screen ">

    <div class="mx-2 h-full overflow-y-auto" id="message-cont">
      <p id="messages"  v-for="(msg, index) in messages" :key="index" :class="[`bg-gray-200 rounded-md w-fit px-4 py-2 m-2`, msg.isSent ? 'ml-auto' : '']"><b>{{ msg.username }}</b>: {{ msg.text }}</p>
    </div>

    <div class="m-2">
      <form @submit.prevent="SendMessage" class="flex items-center space-x-2">
        <input type="text" v-model="messageValue" id="input" name="input" class="border border-black rounded-md p-2 w-full" placeholder="type a message..." autocomplete="off">
        <button type="submit" class="bg-green-500 px-6 py-2 rounded-md">Send</button>
      </form>
    </div>

  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import PieSocket from "piesocket-js";

var channel;
var selfName;
const messageValue = ref('')
const messages = reactive([]);
const username = "user_"+(Math.floor(Math.random() * 1000));
const messageBox = document.getElementById('message-cont');

const pieSocket = new PieSocket({
clusterId: "free.blr2",
apiKey: "2ZilnOus6sDs7od7bbVYQgx8LlgAfabf7yGLJlUt",
notifySelf: true,
presence: true,
userId: username
});

pieSocket.subscribe("chat-room").then((ch) => {
channel = ch;
console.log("Channel is ready");

channel.listen("system:member_joined", function(data){
  console.log(data);  
  
})

channel.listen("new_message", (data, meta) => {
      console.log("New message: ", data);
      if(data.sender != username)
      messages.push({ text: data.message, isSent: false, username: data.sender });
})
})

const SendMessage = () => {

  if(messageValue.value != ''){

    channel.publish("new_message", {
      message: messageValue.value,
      sender: username
    })
    
    messages.push({ text: messageValue.value, isSent: true, username: 'You' });
  }
  messageValue.value = '';
}

</script>