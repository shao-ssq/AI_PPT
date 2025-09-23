<template>
  <div class="chat-container">
    <h2 class="title">AI 智能助手</h2>

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
      <img @click="sendMessage" src="@/assets/send.svg" class="send-img" alt="按钮" />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const userInput = ref('')
const messages = ref([])
const lastRef = ref(null)
const loading = ref(false)

// 发送消息（真实请求后端LLM接口，流式显示回复）
async function sendMessage() {
  if (!userInput.value.trim() || loading.value) return

  messages.value.push({ role: 'user', content: userInput.value })
  const question = userInput.value
  userInput.value = ''
  loading.value = true

  // 新增AI消息（流式追加内容）
  const aiMsg = { role: 'ai', content: '' }
  messages.value.push(aiMsg)

  try {
    const response = await fetch('http://127.0.0.1:5000/tools/aippt_ask', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ question })
    })
    if (!response.body) throw new Error('无响应流')
    const reader = response.body.getReader()
    const decoder = new TextDecoder('utf-8')
    let done = false
      while (!done) {
        const { value, done: doneReading } = await reader.read()
        done = doneReading
        if (value) {
          const chunk = decoder.decode(value)
          aiMsg.content += chunk
          // 强制触发响应式刷新，保证流式内容实时显示
          messages.value = [...messages.value]
          // 实时滚动到底部
          lastRef.value.scrollIntoView({ behavior: 'smooth' })
        }
    }
  } catch (e) {
    aiMsg.content += '\n[AI接口异常] ' + (e.message || e)
  } finally {
    loading.value = false
    setTimeout(() => {
      lastRef.value.scrollIntoView({ behavior: 'smooth' })
    }, 200)
  }
}
</script>

<style scoped>
.chat-container {
  width: 100%;
  margin: 0;
  padding: 0;
  height: calc(100vh - 110px);
  font-family: "Arial", sans-serif;
  display: flex;
  flex-direction: column;
}
.title {
  text-align: center;
  margin-bottom: 12px;
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
  padding: 6px 3px;
  border-radius: 6px;
  border: 1px solid #ccc;
}
.send-img {
  margin-left: 5px;
  width: 35px;
  height: 35px;
}
</style>