<template>
  <div class="chat-container">
    <h2 class="title">
      <svg t="1758768763709" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="8146"
           xmlns:xlink="http://www.w3.org/1999/xlink" width="40" height="40">
        <path d="M175.776914 292.571429h371.741257l35.108572-58.397258h-181.8624c-16.442514 0-18.783086-13.370514-14.628572-21.152914 12.873143-24.049371 39.7312-75.776 80.574172-155.2384 16.676571-23.610514 35.050057-35.401143 55.149714-35.401143 24.341943 0 35.693714 11.790629 34.026057 35.401143l-58.133943 116.9408c106.232686-1.404343 164.571429-1.404343 175.016229 0 8.192 1.082514 11.117714 0.321829 17.6128 10.532572 6.524343 10.210743-0.117029 14.1312 0 18.519771-8.250514 14.511543-12.756114 21.884343-17.6128 30.398171C648.8064 273.934629 635.582171 292.571429 645.12 292.571429h203.893029c11.849143 0 44.763429-3.861943 66.472228 22.293942 14.453029 17.437257 21.357714 39.789714 20.743314 66.998858 0.468114 116.677486 0.702171 184.290743 0.702172 202.839771 0 3.803429 19.0464-69.485714 57.841371-33.850514 17.642057 34.289371 21.328457 163.050057 0 209.832228C965.485714 813.290057 936.930743 729.673143 936.228571 732.511086c0 0 0.702171 109.626514 0.702172 177.005714 0 6.144 4.973714 38.5024-21.445486 62.142171-17.6128 15.798857-39.789714 23.610514-66.472228 23.522743l-703.429486-6.465828c-19.456-2.984229-34.523429-12.170971-45.290057-27.589486-10.708114-15.389257-14.921143-33.850514-12.522057-55.413029v-173.202285c-18.870857 33.938286-38.239086 43.680914-58.046172 29.257143C0 740.205714-3.042743 578.384457 27.735771 556.383086c20.509257-14.687086 40.521143-5.266286 60.035658 28.350171v-202.810514c0-27.326171 9.8304-49.649371 29.461942-66.998857 19.6608-17.378743 39.175314-24.810057 58.514286-22.3232zM175.542857 380.342857v526.628572h672.914286V380.342857H175.542857z"
              fill="#000000" p-id="8147"></path>
        <path d="M365.714286 658.285714m-73.142857 0a73.142857 73.142857 0 1 0 146.285714 0 73.142857 73.142857 0 1 0-146.285714 0Z"
              fill="#000000" p-id="8148"></path>
        <path d="M687.542857 658.285714m-73.142857 0a73.142857 73.142857 0 1 0 146.285714 0 73.142857 73.142857 0 1 0-146.285714 0Z"
              fill="#000000" p-id="8149"></path>
      </svg>
    </h2>


    <div class="chat-window">
      <div
          v-for="(msg, index) in messages"
          :key="index"
          :class="['chat-message', msg.role]"
      >
        <span>{{ msg.content }}</span>
      </div>
      <div ref="lastRef"></div>
    </div>

    <div class="input-area">
      <input
          v-model="userInput"
          type="text"
          placeholder="有什么可以帮你的吗？"
          @keyup.enter="sendMessage"
      />
      <img @click="sendMessage" src="@/assets/send.svg" class="send-img" alt="按钮"/>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

const userInput = ref('')
const messages = ref([])

onMounted(() => {
  const saved = localStorage.getItem('chatMessages')
  if (saved) {
    messages.value = JSON.parse(saved)
  } else {
    messages.value = [
      { role: 'ai', content: '你好，我是你的智能助手~ 有什么可以帮你？' }
    ]
  }
})

// 监听 messages 变化，自动保存
watch(messages, (newVal) => {
  localStorage.setItem('chatMessages', JSON.stringify(newVal))
}, { deep: true })

const lastRef = ref(null)
const loading = ref(false)

// 发送消息（真实请求后端LLM接口，流式显示回复）
async function sendMessage() {
  if (!userInput.value.trim() || loading.value) return

  messages.value.push({role: 'user', content: userInput.value})
  const question = userInput.value
  userInput.value = ''
  loading.value = true

  // 新增AI消息（流式追加内容）
  const aiMsg = {role: 'ai', content: ''}
  messages.value.push(aiMsg)

  try {
    const response = await fetch('http://127.0.0.1:5000/tools/aippt_ask', {
      method: 'POST',
      headers: {'Content-Type': 'application/json'},
      body: JSON.stringify({question})
    })
    if (!response.body) throw new Error('无响应流')
    const reader = response.body.getReader()
    const decoder = new TextDecoder('utf-8')
    let done = false
    while (!done) {
      const {value, done: doneReading} = await reader.read()
      done = doneReading
      if (value) {
        const chunk = decoder.decode(value)
        aiMsg.content += chunk
        // 强制触发响应式刷新，保证流式内容实时显示
        messages.value = [...messages.value]
        // 实时滚动到底部
        lastRef.value.scrollIntoView({behavior: 'smooth'})
      }
    }
  }
  catch (e) {
    aiMsg.content += '\n[AI接口异常] ' + (e.message || e)
  }
  finally {
    loading.value = false
    setTimeout(() => {
      lastRef.value.scrollIntoView({behavior: 'smooth'})
    }, 200)
  }
}
</script>

<style scoped>
.chat-container {
  width: 100%;
  margin: 0;
  padding: 0;
  height: calc(100vh - 130px);
  font-family: "Arial", sans-serif;
  display: flex;
  flex-direction: column;
}

.title {
  text-align: center;
  margin-bottom: 15px;
}

.chat-window {
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.chat-message {
  margin: 8px 0;
  padding: 8px 12px;
  border-radius: 6px;
  max-width: 80%;
  word-wrap: break-word;
}

.chat-message.user {
  background: #e6f7ff;
  align-self: flex-end;
}

.chat-message.ai {
  background: #f6ffed;
}

.input-area {
  display: flex;
  margin-top: 10px;
}

.input-area input {
  flex: 1;
  padding: 7px 7px;
  border-radius: 6px;
  border: 2px solid #3364ae;
}


.input-area input:focus {
  border-image: linear-gradient(135deg, #d897fd, #9a8cfd, #33bcfc) 1;
  outline: none;
  box-shadow: 0 0 8px rgba(217, 151, 253, 0.5);
}

.send-img {
  margin-left: 5px;
  width: 35px;
  height: 35px;
}
</style>