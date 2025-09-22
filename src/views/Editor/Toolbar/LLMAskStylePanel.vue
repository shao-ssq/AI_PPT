<template>
  <div class="chat-container">
    <h2 class="title">AI 智能助手</h2>

    <div class="chat-window">
      <div
          v-for="(msg, index) in messages"
          :key="index"
          :class="['chat-message', msg.role]"
      >
        <strong>{{ msg.role === 'user' ? '用户' : 'AI' }}:</strong>
        <span>{{ msg.content }}</span>
      </div>
    </div>

    <div class="input-area">
      <input
          v-model="userInput"
          type="text"
          placeholder="请输入你的问题..."
          @keyup.enter="sendMessage"
      />
      <button @click="sendMessage">发送</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue"

const userInput = ref("")
const messages = ref([])

// 模拟 LLM 响应
function fakeLLMResponse(question) {
  const answers = [
    "这是个很好的问题，我来详细解释一下……",
    "你可以这样理解：LLM 就像一个超级搜索引擎 + 语言专家。",
    "换句话说，它会基于大量训练数据给出最合理的答案。",
    "我不太确定，但可以给你一些思路……",
  ]
  const randomIndex = Math.floor(Math.random() * answers.length)
  return answers[randomIndex]
}

// 发送消息
function sendMessage() {
  if (!userInput.value.trim()) return

  // 添加用户消息
  messages.value.push({ role: "user", content: userInput.value })
  const question = userInput.value
  userInput.value = ""

  // 模拟 AI 回复（延迟）
  setTimeout(() => {
    const answer = fakeLLMResponse(question)
    messages.value.push({ role: "ai", content: answer })
  }, Math.random() * 1500 + 500)
}
</script>

<style scoped>
.chat-container {
  max-width: 600px;
  margin: 30px auto;
  padding: 20px;
  border-radius: 12px;
  border: 1px solid #ddd;
  background: #fafafa;
  font-family: "Arial", sans-serif;
}
.title {
  text-align: center;
  margin-bottom: 15px;
}
.chat-window {
  height: 400px;
  overflow-y: auto;
  padding: 10px;
  border: 1px solid #eee;
  border-radius: 8px;
  background: #fff;
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
  padding: 8px;
  border-radius: 6px;
  border: 1px solid #ccc;
}
.input-area button {
  margin-left: 8px;
  padding: 8px 16px;
  border: none;
  background: #1677ff;
  color: white;
  border-radius: 6px;
  cursor: pointer;
}
.input-area button:hover {
  background: #4096ff;
}
</style>
