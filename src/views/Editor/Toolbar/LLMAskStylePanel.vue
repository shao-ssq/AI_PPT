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

// 模拟 LLM 响应
function fakeLLMResponse(question) {
  const answers = [
    '这是个很好的问题，我来详细解释一下……',
    '你可以这样理解：LLM 就像一个超级搜索引擎 + 语言专家。',
    '换句话说，它会基于大量训练数据给出最合理的答案。',
    '我不太确定，但可以给你一些思路……',
  ]
  const randomIndex = Math.floor(Math.random() * answers.length)
  return answers[randomIndex]
}

// 发送消息
function sendMessage() {
  if (!userInput.value.trim() || loading.value) return

  // 添加用户消息
  messages.value.push({ role: 'user', content: userInput.value })
  const question = userInput.value
  userInput.value = ''
  loading.value = true

  // 模拟 AI 回复（延迟）
  setTimeout(() => {
    const answer = fakeLLMResponse(question)
    messages.value.push({ role: 'ai', content: answer })
    loading.value = false
  }, Math.random() * 1500 + 500)
  setTimeout(() => {
    lastRef.value.scrollIntoView({ behavior: 'smooth' })
  }, Math.random() * 1500 + 500)
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