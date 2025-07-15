<template>
  <div class="user-center-trigger" @click="$emit('open-user-center')">
    <img :src="userInfo.avatar" alt="用户头像" class="user-avatar" />
  </div>
</template>

<script>
export default {
  name: 'UserCenterTrigger',
  props: {
    userInfo: {
      type: Object,
      required: true,
    },
  },
  emits: ['open-user-center'],
}
</script>

<style scoped>
.user-center-trigger {
  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 10;
  cursor: pointer;
}

.user-avatar {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  border: 3px solid white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  transition: transform 0.3s ease;
}

.user-avatar:hover {
  transform: scale(1.1);
}
</style>