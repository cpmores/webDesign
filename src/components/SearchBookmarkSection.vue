<template>
  <div class="search-bookmark-section">
    <div class="butcontainer">
      <button @click="$emit('search')" class="search-btn">Search</button>
      <button @click="$emit('add-bookmark')" class="bookmark-btn">Bookmark</button>
    </div>
    <!-- 单一输入框，同时用于搜索和收藏 -->
    <div class="search-container">
      <div class="search-input-group">
        <input
          :value="searchQuery"
          @input="$emit('update:searchQuery', $event.target.value); $emit('update:bookmarkUrl', $event.target.value); $emit('search-input')"
          @focus="$emit('search-focus')"
          @blur="$emit('hide-suggestions')"
          type="text"
          placeholder="输入网址、标题、标签进行搜索或添加收藏..."
          class="search-input"
        />
      </div>

      <!-- 搜索排序选择 -->
      <div class="search-sort-container">
        <label for="search-sort" class="sort-label">SortBy：</label>
        <select
          id="search-sort"
          :value="searchSortBy"
          @change="$emit('update:searchSortBy', $event.target.value)"
          class="search-sort-select"
        >
          <option value="time">Time</option>
          <option value="click_count">ClickRate</option>
        </select>
      </div>

      <!-- 搜索建议和历史记录 -->
      <div
        v-if="showSuggestions && (prefixMatchResults.length > 0 || (showHistory && searchHistory.length > 0))"
        class="search-suggestions"
      >
        <!-- 前缀匹配结果 -->
        <div v-if="prefixMatchResults.length > 0">
          <div class="suggestions-header">
            <span class="suggestions-title">匹配结果</span>
            <span class="suggestions-count">({{ prefixMatchResults.length }})</span>
          </div>
          <div
            v-for="result in prefixMatchResults"
            :key="result"
            @click="$emit('select-suggestion', result)"
            class="suggestion-item"
          >
            {{ result }}
          </div>
        </div>

        <!-- 搜索历史记录 -->
        <div v-if="showHistory && searchHistory.length > 0">
          <div class="suggestions-header">
            <span class="suggestions-title">搜索历史</span>
            <span class="suggestions-count">({{ searchHistory.length }})</span>
          </div>
          <div
            v-for="historyItem in searchHistory"
            :key="historyItem"
            @click="$emit('select-suggestion', historyItem)"
            class="suggestion-item history-item"
          >
            <span class="history-icon">🕒</span>
            {{ historyItem }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SearchBookmarkSection',
  props: {
    searchQuery: String,
    searchSortBy: String,
    prefixMatchResults: Array,
    searchHistory: Array,
    showSuggestions: Boolean,
    showHistory: Boolean,
    bookmarkUrl: String,
  },
  emits: [
    'update:searchQuery',
    'update:searchSortBy',
    'update:bookmarkUrl',
    'search',
    'search-input',
    'search-focus',
    'hide-suggestions',
    'select-suggestion',
    'add-bookmark',
  ],
}
</script>

<style scoped>
.search-bookmark-section {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  text-align: center;
}

.search-container {
  position: relative;
  margin-bottom: 30px;
  display: flex;
}

.search-input-group {
  display: flex;
  flex: 1 1 auto;
  gap: 10px;
  margin-bottom: 15px;
}

.search-sort-container {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-shrink: 0;
  margin-right: 10px;
  font-size: 1.1rem;
  font-weight: 600;
  font-family: 'ReadexPro', sans-serif;
}

.search-input {
  padding: 4vh 20px;
  border: none;
  width: 100%;
  font-size: 1.5rem;
  /* box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1); */
  background: transparent;
  outline: none;
}

.search-btn {
  position: absolute;
  left: 3px;
  top: 3px;
  padding: 10px 20px;
  background: transparent;
  color: black;
  border: none;
  /* border-radius: 20px; */
  cursor: pointer;
  font-size: 1.2rem;
  z-index: 998;
  font-family: 'ReadexPro', sans-serif;
  transition: background 0.3s ease;
}

.butcontainer {
    display: flex;
}

.search-btn:hover {
  background: #000;
  color: whitesmoke;
}

.search-sort-select {
  padding: 8px 12px;
  border: 1px solid grey;
  /* border-radius: 6px; */
  background: transparent;
  color: #3E4043;
  cursor: pointer;
  outline: none;
  font-size: 1rem;
  font-weight: 600;
  font-family: 'ReadexPro', sans-serif;
  transition: border-color 0.3s ease;
}

.search-sort-select:hover {
  border-color: whitesmoke;
}

.search-sort-select:focus {
  border-color: whitesmoke;
  box-shadow: 0 0 0 2px rgba(74, 144, 226, 0.2);
}


.search-suggestions {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  z-index: 100;
  margin-top: 5px;
  max-height: 300px;
  overflow-y: auto;
}

.suggestions-header {
  padding: 10px 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
  border-radius: 8px 8px 0 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.suggestions-title {
  font-weight: 600;
  color: #495057;
  font-size: 14px;
}

.suggestions-count {
  color: #6c757d;
  font-size: 12px;
  background: #e9ecef;
  padding: 2px 8px;
  border-radius: 12px;
}

.suggestion-item {
  padding: 12px 20px;
  cursor: pointer;
  border-bottom: 1px solid #f0f0f0;
  transition: background 0.3s ease;
}

.suggestion-item:hover {
  background: #f8f9fa;
}

.tag-suggestion {
  color: #ff9800;
  font-weight: 500;
}

.tag-suggestion::before {
  content: '🏷️ ';
  margin-right: 5px;
}

.suggestion-item:last-child {
  border-bottom: none;
}

.history-item {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #666;
}

.history-icon {
  font-size: 14px;
  opacity: 0.7;
}

.bookmark-container {
  display: flex;
  gap: 10px;
}

.bookmark-input {
  flex: 1;
  padding: 15px 20px;
  border: none;
  border-radius: 25px;
  font-size: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  outline: none;
}

.search-btn {
  position: absolute;
  left: 1ex;
  top: 3px;
  padding: 10px 20px;
  background: transparent;
  color: black;
  border: none;
  /* border-radius: 20px; */
  cursor: pointer;
  font-size: 1.2rem;
  z-index: 998;
  font-family: 'ReadexPro', sans-serif;
  transition: background 0.3s ease;
}

.bookmark-btn {
  position: absolute;
  padding: 10px 20px;
  left: 12ex;
  top: 3px;
  background: transparent;
  color: #722F37;
  border: none;
  /* border-radius: 25px; */
  cursor: pointer;
  font-size: 1.2rem;
  z-index: 998;
  font-family: 'ReadexPro', sans-serif;
  transition: background 0.3s ease;
}

.bookmark-btn:hover {
  background: #000;
  color: whitesmoke
}
</style>