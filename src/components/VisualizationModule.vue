<template>
  <div class="visualization-module">
    <h2>数据可视化</h2>
    <div class="visualization-content">
      <div class="chart-section">
        <h3>标签词云</h3>
        <p class="chart-description">展示您收藏中使用的所有标签，字体大小代表使用频率</p>
        <WordCloud :tags="userTags" :tag-counts="tagCounts" />
      </div>
      <div class="stats-section">
        <div class="stat-item">
          <div class="stat-number">{{ totalBookmarksCount }}</div>
          <div class="stat-label">总收藏数</div>
        </div>
        <div class="stat-item">
          <div class="stat-number">{{ userTags.length }}</div>
          <div class="stat-label">标签种类</div>
        </div>
        <div class="stat-item">
          <div class="stat-number">{{ mostUsedTag }}</div>
          <div class="stat-label">最常用标签</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import WordCloud from './WordCloud.vue'

export default {
  name: 'VisualizationModule',
  components: { WordCloud },
  props: {
    userTags: Array,
    tagCounts: Object,
    totalBookmarksCount: Number,
    mostUsedTag: String,
  },
}
</script>

<style scoped>
.visualization-module {
  text-align: center;
  padding: 40px 20px;
  background: #f8f9fa;
  border-radius: 12px;
  margin-bottom: 40px;
}

.visualization-module h2 {
  color: #333;
  margin-bottom: 30px;
  font-size: 24px;
}

.visualization-content {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.chart-section {
  text-align: center;
}

.chart-section h3 {
  color: #333;
  margin-bottom: 10px;
  font-size: 18px;
}

.chart-description {
  color: #666;
  margin-bottom: 20px;
  font-size: 14px;
}

.stats-section {
  display: flex;
  justify-content: center;
  gap: 40px;
  flex-wrap: wrap;
}

.stat-item {
  text-align: center;
  padding: 20px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  min-width: 120px;
  transition: transform 0.3s ease;
}

.stat-item:hover {
  transform: translateY(-2px);
}

.stat-number {
  font-size: 32px;
  font-weight: bold;
  color: #4a90e2;
  margin-bottom: 8px;
}

.stat-label {
  color: #666;
  font-size: 14px;
}
</style>