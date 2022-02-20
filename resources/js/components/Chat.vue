<template>
  <div class="chat card" ref="hasScrolledToBottom">
    <div class="card-body scrollable">
      <template v-for="message in messages">
        <div
          class="message message-receive"
          v-if="user.id != message.user.id"
          :key="message.id"
        >
          <p>
            <strong class="primary-font"> {{ message.user.name }} </strong>
            {{ message.message }}
          </p>
        </div>
        <div class="message message-send" v-else :key="message.id">
          <strong class="primary-font"> {{ message.user.name }} </strong>
          {{ message.message }}
        </div>
      </template>
    </div>
    <div class="chat-form input-group">
      <input
        id="btn-input"
        type="text"
        name="message"
        class="form-control input-sm message-"
        placeholder="Type your message here..."
        v-model="newMessage"
        @keyup.enter="addMessage"
      />
      <span class="input-group-btn">
        <button class="btn btn-primary" id="btn-chat" @click="addMessage">
          Send
        </button>
      </span>
    </div>
  </div>
</template>

<script>
import { reactive, onMounted, ref, onUpdated } from "vue";
import axios from "axios";
export default {
  name: "Chat",
  props: {
    user: {
      type: Object,
      required: true,
    },
  },
  setup(props) {
    let messages = ref([]);
    let newMessage = ref("");
    let hasScrolledToBottom = ref("");

    onMounted(() => {
      fetchMessages();
    });

    onUpdated(() => {
      scrollBottom();
    });

    Echo.private("chat-channel").listen("SendMessage", (e) => {
      messages.value.push({
        message: e.message.message,
        user: e.user,
      });
    });

    const fetchMessages = async () => {
      await axios.get("/messages").then((response) => {
        messages.value = response.data;
      });
    };

    const addMessage = async () => {
      let userMessage = {
        user: props.user,
        message: newMessage.value,
      };
      messages.value.push(userMessage);
      await axios.post("/messages", userMessage).then((response) => {
        console.log(response);
      });
      newMessage.value = "";
    };

    const scrollBottom = () => {
      if (messages.value.length > 1) {
        let el = hasScrolledToBottom.value;
        el.scrollTop = el.scrollHeight;
      }
    };

    return {
      messages,
      newMessage,
      fetchMessages,
      addMessage,
      hasScrolledToBottom,
    };
  },
};
</script>