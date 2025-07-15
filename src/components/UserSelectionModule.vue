<template>
  <div class="user-selection-module">
    <h3>标签筛选</h3>
    <div v-if="userTags.length === 0" class="no-tags">
      <p>暂无标签，添加收藏时可以为收藏添加标签</p>
    </div>
    <div v-else class="tags-container">
      <button @click="$emit('select-all')" :class="['tag-btn', { active: selectedTag === '' }]">
        全部 ({{ totalBookmarksCount }})
      </button>
      <button
        v-for="tag in userTags"
        :key="tag"
        @click="$emit('select-tag', tag)"
        :class="['tag-btn', { active: selectedTag === tag }]"
        :title="`点击查看包含「${tag}」标签的收藏`"
      >
        {{ tag }} ({{ tagCounts[tag] || 0 }})
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserSelectionModule',
  props: {
    userTags: Array,
    tagCounts: Object,
    selectedTag: String,
    totalBookmarksCount: Number,
  },
  emits: ['select-all', 'select-tag'],
}
</script>

<style scoped>
.user-selection-module {
  margin-bottom: 40px;
}

.user-selection-module h3 {
  color: #333;
  margin-bottom: 20px;
}

.no-tags {
  text-align: center;
  padding: 20px;
  color: #666;
  background: #f8f9fa;
  border-radius: 8px;
  border: 1px dashed #dee2e6;
}

.no-tags p {
  margin: 0;
  font-size: 14px;
}

.tags-container {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.tag-btn {
  padding: 8px 16px;
  background: #f8f9fa;
  color: #495057;
  border: 2px solid #e9ecef;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 14px;
  font-weight: 500;
  position: relative;
  overflow: hidden;
}

.tag-btn:hover {
  background: #e9ecef;
  border-color: #4a90e2;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(74, 144, 226, 0.2);
}

.tag-btn.active {
  background: #4a90e2;
  color: white;
  border-color: #4a90e2;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(74, 144, 226, 0.3);
}

.tag-btn::after {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s;
}

.tag-btn:hover::after {
  left: 100%;
}
</style>