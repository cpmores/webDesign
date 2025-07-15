<template>
  <div>
    <div ref="aiBall" class="ai-assistant-ball" @mousedown="startDrag" @click="$emit('update:show', true)">
      🤖
    </div>
    <div v-if="show" class="ai-chat-overlay" @click="$emit('update:show', false)">
      <div class="ai-chat-modal" @click.stop>
        <div class="ai-chat-header">
          <h3>AI助手</h3>
          <button @click="$emit('update:show', false)" class="close-btn">×</button>
        </div>
        <div class="ai-chat-messages" ref="chatMessages">
          <div v-for="message in chatMessages" :key="message.id" :class="['message', message.type]">
            <div class="message-content">{{ message.content }}</div>
            <div class="message-time">{{ formatTime(message.timestamp) }}</div>
          </div>
        </div>
        <div class="ai-chat-input">
          <input
            :value="aiInput"
            @input="$emit('update:aiInput', $event.target.value)"
            @keyup.enter="$emit('send-ai-message')"
            type="text"
            placeholder="输入您的问题..."
            class="ai-input"
          />
          <button @click="$emit('send-ai-message')" class="ai-send-btn">提交</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AIAssistant',
  props: {
    show: Boolean,
    aiInput: String,
    chatMessages: Array,
  },
  emits: ['update:show', 'update:aiInput', 'send-ai-message'],
  data() {
    return {
      isDragging: false,
      dragOffset: { x: 0, y: 0 },
    }
  },
  mounted() {
    this.initAIBallPosition()
  },
  methods: {
    initAIBallPosition() {
      const ball = this.$refs.aiBall
      if (ball) {
        ball.style.right = '20px'
        ball.style.bottom = '20px'
      }
    },
    startDrag(event) {
      event.preventDefault()
      this.isDragging = true
      const ball = this.$refs.aiBall
      const rect = ball.getBoundingClientRect()
      this.dragOffset = {
        x: event.clientX - rect.left,
        y: event.clientY - rect.top,
      }
      document.addEventListener('mousemove', this.onDrag)
      document.addEventListener('mouseup', this.stopDrag)
    },
    onDrag(event) {
      if (!this.isDragging) return
      const ball = this.$refs.aiBall
      const x = event.clientX - this.dragOffset.x
      const y = event.clientY - this.dragOffset.y
      ball.style.left = x + 'px'
      ball.style.top = y + 'px'
      ball.style.right = 'auto'
      ball.style.bottom = 'auto'
    },
    stopDrag() {
      this.isDragging = false
      document.removeEventListener('mousemove', this.onDrag)
      document.removeEventListener('mouseup', this.stopDrag)
    },
    formatTime(timestamp) {
      const date = new Date(timestamp)
      return date.toLocaleTimeString('zh-CN')
    },
    scrollToBottom() {
      const chatMessages = this.$refs.chatMessages
      if (chatMessages) {
        chatMessages.scrollTop = chatMessages.scrollHeight
      }
    },
  },
}
</script>

<style scoped>
.ai-assistant-ball {
  position: fixed;
  width: 60px;
  height: 60px;
  background: #4a90e2;
  color: white;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 24px;
  cursor: pointer;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  z-index: 999;
  transition: transform 0.3s ease;
  user-select: none;
}

.ai-assistant-ball:hover {
  transform: scale(1.1);
}

.ai-chat-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1001;
}

.ai-chat-modal {
  background: white;
  border-radius: 12px;
  width: 500px;
  max-width: 90vw;
  height: 600px;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.ai-chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #e9ecef;
}

.ai-chat-header h3 {
  margin: 0;
  color: #333;
}

.ai-chat-messages {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.message {
  max-width: 80%;
  padding: 12px 16px;
  border-radius: 12px;
  word-wrap: break-word;
}

.message.user {
  align-self: flex-end;
  background: #4a90e2;
  color: white;
}

.message.ai {
  align-self: flex-start;
  background: #f8f9fa;
  color: #333;
}

.message-content {
  margin-bottom: 5px;
}

.message-time {
  font-size: 12px;
  opacity: 0.7;
}

.ai-chat-input {
  display: flex;
  gap: 10px;
  padding: 20px;
  border-top: 1px solid #e9ecef;
}

.ai-input {
  flex: 1;
  padding: 12px 16px;
  border: 1px solid #e1e5e9;
  border-radius: 8px;
  outline: none;
}

.ai-input:focus {
  border-color: #4a90e2;
}

.ai-send-btn {
  padding: 12px 20px;
  background: #4a90e2;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s ease;
}

.ai-send-btn:hover {
  background: #357abd;
}
</style>