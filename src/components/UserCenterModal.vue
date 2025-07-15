<template>
  <div v-if="show" class="user-center-overlay" @click="$emit('update:show', false)">
    <div class="user-center-modal" @click.stop>
      <div class="user-center-header">
        <h2>用户中心</h2>
        <button @click="$emit('update:show', false)" class="close-btn">返回</button>
      </div>
      <div class="user-center-content">
        <img :src="userInfo.avatar" alt="用户头像" class="user-center-avatar" />
        <div class="user-info">
          <p class="user-id">用户ID: {{ userInfo.id }}</p>
          <p class="user-name">用户名: {{ userInfo.username }}</p>
          <p class="user-email">邮箱: {{ userInfo.email }}</p>
        </div>
        <button @click="$emit('logout')" class="logout-btn">退出登录</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserCenterModal',
  props: {
    show: Boolean,
    userInfo: Object,
  },
  emits: ['update:show', 'logout'],
}
</script>

<style scoped>
.user-center-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.user-center-modal {
  background: white;
  border-radius: 12px;
  padding: 30px;
  width: 400px;
  max-width: 90vw;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.user-center-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
}

.user-center-header h2 {
  color: #333;
  margin: 0;
}

.close-btn {
  background: #f8f9fa;
  color: #6c757d;
  border: 1px solid #e1e5e9;
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.close-btn:hover {
  background: #e9ecef;
}

.user-center-content {
  text-align: center;
}

.user-center-avatar {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  margin-bottom: 20px;
}

.user-info p {
  margin: 10px 0;
  color: #666;
}

.logout-btn {
  margin-top: 20px;
  padding: 10px 20px;
  background: #dc3545;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.3s ease;
}

.logout-btn:hover {
  background: #c82333;
}
</style>