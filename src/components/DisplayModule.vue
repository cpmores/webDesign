<template>
  <div class="display-module">
    <div class="display-header">
      <h3>
        收藏列表
        <span v-if="selectedTag" class="filter-info"> (筛选: {{ selectedTag }}) </span>
        <span v-if="searchQuery && !selectedTag" class="search-info">
          (搜索: {{ searchQuery }})
        </span>
      </h3>
      <div v-if="!selectedTag && !searchQuery" class="sort-controls">
        <span class="sort-label">排序方式：</span>
        <button
          @click="$emit('change-sort', 'time')"
          :class="['sort-btn', { active: currentSortBy === 'time' }]"
        >
          按时间
        </button>
        <button
          @click="$emit('change-sort', 'click_count')"
          :class="['sort-btn', { active: currentSortBy === 'click_count' }]"
        >
          按点击次数
        </button>
      </div>
    </div>
    <div v-if="bookmarks.length === 0" class="no-bookmarks">
      <p v-if="selectedTag">没有找到包含"{{ selectedTag }}"标签的收藏</p>
      <p v-else-if="searchQuery">没有找到包含"{{ searchQuery }}"的收藏</p>
      <p v-else>暂无收藏内容</p>
    </div>
    <div v-else class="bookmarks-list">
      <div v-for="bookmark in bookmarks" :key="bookmark.id" class="bookmark-item">
        <div class="bookmark-content">
          <div class="bookmark-header">
            <h4 class="bookmark-title">{{ bookmark.title }}</h4>
            <div class="bookmark-actions">
              <div class="bookmark-click-count">
                <span class="click-icon">👆</span>
                <span class="click-number">{{ bookmark.clickCount || 0 }}</span>
              </div>
              <button
                @click="$emit('delete-bookmark', bookmark)"
                class="delete-btn"
                title="删除收藏"
              >
                🗑️
              </button>
            </div>
          </div>
          <a
            :href="bookmark.url"
            class="bookmark-url"
            @click="$emit('bookmark-click', bookmark, $event)"
          >
            {{ bookmark.url }}
          </a>
          <div class="bookmark-tags">
            <span v-for="tag in bookmark.tags" :key="tag" class="bookmark-tag">
              {{ tag }}
            </span>
          </div>
          <div class="bookmark-info">
            <div class="bookmark-date">
              <span class="info-label">收藏时间：</span>
              <span class="info-value">{{ formatDate(bookmark.createdAt) }}</span>
            </div>
            <div class="bookmark-clicks">
              <span class="info-label">点击次数：</span>
              <span class="info-value">{{ bookmark.clickCount || 0 }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-if="totalPages > 1" class="pagination">
      <button
        @click="$emit('change-page', currentPage - 1)"
        :disabled="currentPage === 1"
        class="page-btn"
      >
        上一页
      </button>
      <span class="page-info">{{ currentPage }} / {{ totalPages }}</span>
      <button
        @click="$emit('change-page', currentPage + 1)"
        :disabled="currentPage === totalPages"
        class="page-btn"
      >
        下一页
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DisplayModule',
  props: {
    bookmarks: Array,
    selectedTag: String,
    searchQuery: String,
    currentSortBy: String,
    currentPage: Number,
    totalPages: Number,
  },
  emits: ['change-sort', 'change-page', 'bookmark-click', 'delete-bookmark'],
  methods: {
    formatDate(dateString) {
      const date = new Date(dateString)
      return date.toLocaleDateString('zh-CN')
    },
  },
}
</script>

<style scoped>
.display-module h3 {
  color: #333;
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.filter-info {
  color: #4a90e2;
  font-size: 14px;
  font-weight: normal;
  background: #e3f2fd;
  padding: 4px 8px;
  border-radius: 12px;
  border: 1px solid #bbdefb;
}

.search-info {
  color: #ff9800;
  font-size: 14px;
  font-weight: normal;
  background: #fff3e0;
  padding: 4px 8px;
  border-radius: 12px;
  border: 1px solid #ffcc02;
}

.no-bookmarks {
  text-align: center;
  padding: 40px;
  color: #666;
}

.bookmarks-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.bookmark-item {
  background: white;
  border: 1px solid #e9ecef;
  border-radius: 8px;
  padding: 20px;
  transition: box-shadow 0.3s ease;
}

.bookmark-item:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.bookmark-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 8px;
}

.bookmark-title {
  color: #333;
  font-size: 18px;
  margin: 0;
  flex: 1;
  margin-right: 15px;
}

.bookmark-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.bookmark-click-count {
  display: flex;
  align-items: center;
  gap: 4px;
  background: #f8f9fa;
  padding: 4px 8px;
  border-radius: 12px;
  border: 1px solid #e9ecef;
  font-size: 12px;
  color: #666;
  white-space: nowrap;
}

.delete-btn {
  background: #dc3545;
  color: white;
  border: none;
  border-radius: 50%;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s ease;
  opacity: 0.7;
}

.delete-btn:hover {
  background: #c82333;
  opacity: 1;
  transform: scale(1.1);
}

.click-icon {
  font-size: 14px;
}

.click-number {
  font-weight: 600;
  color: #4a90e2;
}

.bookmark-url {
  color: #4a90e2;
  text-decoration: none;
  display: block;
  margin-bottom: 10px;
  word-break: break-all;
}

.bookmark-url:hover {
  text-decoration: underline;
}

.bookmark-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  margin-bottom: 10px;
}

.bookmark-tag {
  background: #e3f2fd;
  color: #1976d2;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
}

.bookmark-info {
  display: flex;
  flex-direction: column;
  gap: 5px;
  margin-top: 10px;
  padding: 10px;
  background: #f8f9fa;
  border-radius: 6px;
  border: 1px solid #e9ecef;
}

.bookmark-date,
.bookmark-clicks {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
}

.info-label {
  color: #666;
  font-weight: 500;
  min-width: 70px;
}

.info-value {
  color: #333;
  font-weight: 600;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  margin-top: 30px;
}

.page-btn {
  padding: 8px 16px;
  background: #4a90e2;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.3s ease;
}

.page-btn:hover:not(:disabled) {
  background: #357abd;
}

.page-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.page-info {
  color: #666;
  font-size: 14px;
}

.display-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  flex-wrap: wrap;
  gap: 15px;
}

.display-header h3 {
  margin: 0;
  color: #333;
}

.sort-controls {
  display: flex;
  align-items: center;
  gap: 10px;
}

.sort-label {
  color: #666;
  font-size: 14px;
  font-weight: 500;
}

.sort-btn {
  padding: 6px 12px;
  background: #f8f9fa;
  color: #666;
  border: 1px solid #e1e5e9;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  transition: all 0.3s ease;
}

.sort-btn:hover {
  background: #e9ecef;
  transform: translateY(-1px);
}

.sort-btn.active {
  background: #4a90e2;
  color: white;
  border-color: #4a90e2;
}
</style>