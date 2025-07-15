<template>
  <div class="main-view">
    <div class="main-interface">
      <UserCenterTrigger :user-info="userInfo" @open-user-center="showUserCenter = true" />
      <SearchBookmarkSection
        v-model:searchQuery="searchQuery"
        v-model:searchSortBy="searchSortBy"
        v-model:bookmarkUrl="bookmarkUrl"
        :prefix-match-results="prefixMatchResults"
        :search-history="searchHistory"
        :show-suggestions="showSuggestions"
        :show-history="showHistory"
        @search="handleSearch"
        @search-input="handleSearchInput"
        @search-focus="handleSearchFocus"
        @hide-suggestions="hideSuggestions"
        @select-suggestion="selectSuggestion"
        @add-bookmark="handleAddBookmark"
      />
    </div>
    <div class="sub-interface" :style="{ top: isSubInterfaceAdjusted ? '10vh' : '0vh' }">
      <button class="toggle-interface-btn" @click="toggleSubInterface">
        {{ !isSubInterfaceAdjusted ? 'Search?' : 'Fold' }}
      </button>
      <VisualizationModule
        :user-tags="userTags"
        :tag-counts="tagCounts"
        :total-bookmarks-count="totalBookmarksCount"
        :most-used-tag="getMostUsedTag()"
      />
      <UserSelectionModule
        :user-tags="userTags"
        :tag-counts="tagCounts"
        :selected-tag="selectedTag"
        :total-bookmarks-count="totalBookmarksCount"
        @select-all="selectAllBookmarks"
        @select-tag="selectTag"
      />
      <DisplayModule
        :bookmarks="bookmarks"
        :selected-tag="selectedTag"
        :search-query="searchQuery"
        :current-sort-by="currentSortBy"
        :current-page="currentPage"
        :total-pages="totalPages"
        @change-sort="changeSortBy"
        @change-page="changePage"
        @bookmark-click="handleBookmarkClick"
        @delete-bookmark="handleDeleteBookmark"
      />
    </div>
    <UserCenterModal
      v-model:show="showUserCenter"
      :user-info="userInfo"
      @logout="handleLogout"
    />
    <AIAssistant
      v-model:show="showAIChat"
      v-model:aiInput="aiInput"
      :chat-messages="chatMessages"
      @send-ai-message="sendAIMessage"
    />
    <TagManager
      :show-tag-modal="showTagModal"
      :bookmark-url="bookmarkUrl"
      @close="closeTagModal"
      @bookmark-added="onBookmarkAdded"
    />
  </div>
</template>

<script>
import {
  getAllBookmarks,
  getBookmarksByTag,
  getUserTags,
  chatWithAI,
  logout,
  recordBookmarkClick,
  deleteBookmark,
  prefixMatch,
  getSearchHistory,
  multiSearchBookmarks,
} from '../services/api.js'
import UserCenterTrigger from '../components/UserCenterTrigger.vue'
import SearchBookmarkSection from '../components/SearchBookmarkSection.vue'
import VisualizationModule from '../components/VisualizationModule.vue'
import UserSelectionModule from '../components/UserSelectionModule.vue'
import DisplayModule from '../components/DisplayModule.vue'
import UserCenterModal from '../components/UserCenterModal.vue'
import AIAssistant from '../components/AIAssistant.vue'
import TagManager from '../components/TagManager.vue'

export default {
  name: 'MainView',
  components: {
    UserCenterTrigger,
    SearchBookmarkSection,
    VisualizationModule,
    UserSelectionModule,
    DisplayModule,
    UserCenterModal,
    AIAssistant,
    TagManager,
  },
  data() {
    return {
      userInfo: { id: '', email: '', avatar: '' },
      showUserCenter: false,
      searchQuery: '',
      searchSuggestions: [],
      prefixMatchResults: [],
      showSuggestions: false,
      searchSortBy: 'time',
      searchHistory: [],
      showHistory: false,
      bookmarkUrl: '',
      bookmarks: [],
      userTags: [],
      tagCounts: {},
      selectedTag: '',
      showTagModal: false,
      totalBookmarksCount: 0,
      currentPage: 1,
      totalPages: 1,
      pageSize: 10,
      currentSortBy: 'time',
      showAIChat: false,
      aiInput: '',
      chatMessages: [],
      isSubInterfaceAdjusted: false, // 新增状态控制 sub-interface 位置
    }
  },
  async mounted() {
    const userData = localStorage.getItem('userData')
    if (userData) {
      await this.initializeWithData(JSON.parse(userData))
    } else {
      const userInfo = localStorage.getItem('user_info')
      if (userInfo) {
        this.userInfo = JSON.parse(userInfo)
        await this.loadUserTags()
        await this.loadAllBookmarks()
      } else {
        localStorage.removeItem('isLoggedIn')
        localStorage.removeItem('userEmail')
        localStorage.removeItem('userId')
        localStorage.removeItem('userData')
        localStorage.removeItem('user_info')
        localStorage.removeItem('auth_token')
        window.dispatchEvent(new CustomEvent('loginStatusChanged'))
      }
    }
  },
  methods: {
    // 新增方法：切换 sub-interface 位置
    toggleSubInterface() {
      this.isSubInterfaceAdjusted = !this.isSubInterfaceAdjusted
    },
    async initializeWithData(userData) {
      this.userInfo = userData.user
      this.bookmarks = userData.bookmarks
      this.totalBookmarksCount = userData.totalBookmarks
      this.userTags = userData.tags
      this.tagCounts = userData.tagCounts
      this.totalPages = Math.ceil(userData.totalBookmarks / this.pageSize)
      console.log('使用初始化数据完成界面初始化')
    },
    async loadUserTags() {
      try {
        let userId = null
        const userData = localStorage.getItem('userData')
        if (userData) {
          const parsedUserData = JSON.parse(userData)
          userId = parsedUserData.user?.userId || parsedUserData.user?.id
        }
        if (!userId) {
          const userInfo = localStorage.getItem('user_info')
          if (userInfo) {
            const parsedUserInfo = JSON.parse(userInfo)
            userId = parsedUserInfo.userId || parsedUserInfo.id
          }
        }
        if (!userId) {
          console.error('无法获取用户ID，跳过加载用户标签')
          return
        }
        const response = await getUserTags(userId)
        if (response.success) {
          this.userTags = response.tags
          this.tagCounts = response.tagCounts || {}
        } else {
          console.error('获取用户标签失败:', response.message)
        }
      } catch (error) {
        console.error('加载用户标签失败:', error)
      }
    },
    async handleSearchFocus() {
      this.showSuggestions = true
      if (!this.searchQuery.trim()) {
        await this.loadSearchHistory()
      }
    },
    async loadSearchHistory() {
      try {
        const response = await getSearchHistory('time')
        if (response.success) {
          this.searchHistory = response.data.queries
          this.showHistory = true
          console.log('搜索历史加载成功:', this.searchHistory)
        }
      } catch (error) {
        console.error('加载搜索历史失败:', error)
        this.searchHistory = []
        this.showHistory = false
      }
    },
    async handleSearchInput() {
      if (this.searchQuery.trim()) {
        this.showHistory = false
        try {
          let userId = null
          const userData = localStorage.getItem('userData')
          if (userData) {
            const parsedUserData = JSON.parse(userData)
            userId = parsedUserData.user?.userId || parsedUserData.user?.id
          }
          if (!userId) {
            const userInfo = localStorage.getItem('user_info')
            if (userInfo) {
              const parsedUserInfo = JSON.parse(userInfo)
              userId = parsedUserInfo.userId || parsedUserInfo.id
            }
          }
          if (!userId) {
            console.error('无法获取用户ID，跳过前缀匹配')
            return
          }
          const response = await prefixMatch(userId, this.searchQuery)
          if (response.success) {
            this.prefixMatchResults = response.data.results
            this.searchSuggestions = this.prefixMatchResults
            console.log('前缀匹配结果:', response.data.results)
            console.log('检测到的语言:', response.data.language)
          }
        } catch (error) {
          console.error('前缀匹配失败:', error)
          this.prefixMatchResults = []
          this.searchSuggestions = []
        }
      } else {
        this.prefixMatchResults = []
        this.searchSuggestions = []
        await this.loadSearchHistory()
      }
    },
    hideSuggestions() {
      setTimeout(() => {
        this.showSuggestions = false
      }, 200)
    },
    selectSuggestion(suggestion) {
      this.searchQuery = suggestion
      this.showSuggestions = false
      if (suggestion.startsWith('标签: ')) {
        const tag = suggestion.replace('标签: ', '')
        this.searchQuery = tag
      }
      this.handleSearch()
    },
    async handleSearch() {
      try {
        const searchParams = {
          keyword: this.searchQuery.trim(),
          sortBy: this.searchSortBy,
        }
        if (this.selectedTag && this.selectedTag !== '') {
          searchParams.tag = this.selectedTag
        }
        if (!searchParams.keyword && !searchParams.tag) {
          if (this.selectedTag) {
            await this.selectTag(this.selectedTag)
          } else {
            await this.loadAllBookmarks()
          }
          return
        }
        const response = await multiSearchBookmarks(searchParams)
        if (response.success) {
          this.bookmarks = response.data
          console.log('搜索成功，结果数量:', this.bookmarks.length)
          this.scrollToDisplayModule()
        }
      } catch (error) {
        console.error('搜索失败:', error)
        this.showErrorMessage('搜索失败，请稍后重试')
      }
    },
    handleAddBookmark() {
      if (!this.bookmarkUrl.trim()) {
        alert('请输入要收藏的网页链接')
        return
      }
      try {
        new URL(this.bookmarkUrl)
      } catch (error) {
        alert('请输入有效的网页链接')
        return
      }
      this.showTagModal = true
    },
    closeTagModal() {
      this.showTagModal = false
      this.bookmarkUrl = ''
    },
    async onBookmarkAdded() {
      await this.loadUserTags()
      if (this.selectedTag) {
        await this.selectTag(this.selectedTag)
      } else {
        await this.loadAllBookmarks()
      }
      this.showSuccessMessage('收藏添加成功！')
    },
    async selectAllBookmarks() {
      this.selectedTag = ''
      this.currentPage = 1
      await this.loadAllBookmarks()
    },
    async changeSortBy(sortBy) {
      if (this.currentSortBy === sortBy) return
      this.currentSortBy = sortBy
      if (!this.selectedTag) {
        await this.loadAllBookmarks()
      }
    },
    async selectTag(tag) {
      this.selectedTag = tag
      await this.loadBookmarksByTag(tag)
    },
    async loadAllBookmarks() {
      try {
        const response = await getAllBookmarks(this.currentSortBy)
        if (response.success) {
          const allBookmarks = []
          let totalCount = 0
          response.data.forEach((group) => {
            if (group.bookmarks && Array.isArray(group.bookmarks)) {
              group.bookmarks.forEach((bookmark) => {
                allBookmarks.push({
                  ...bookmark,
                  id: `${bookmark.url}_${bookmark.tag}`,
                  title: bookmark.url,
                  tags: [bookmark.tag],
                  clickCount: bookmark.click_count,
                  createdAt: bookmark.created_at,
                })
              })
              totalCount += group.bookmarks.length
            }
          })
          this.bookmarks = allBookmarks
          this.totalBookmarksCount = totalCount
          this.totalPages = 1
        }
      } catch (error) {
        console.error('加载所有收藏失败:', error)
        this.showErrorMessage('加载收藏失败，请稍后重试')
      }
    },
    async loadBookmarksByTag(tag) {
      try {
        const response = await getBookmarksByTag(tag)
        if (response.success) {
          const bookmarks = response.data.map((bookmark) => ({
            ...bookmark,
            id: `${bookmark.url}_${bookmark.tag}`,
            title: bookmark.url,
            tags: [bookmark.tag],
            clickCount: bookmark.click_count,
            createdAt: bookmark.created_at,
          }))
          this.bookmarks = bookmarks
          this.totalBookmarksCount = bookmarks.length
          this.totalPages = 1
        }
      } catch (error) {
        console.error('加载收藏失败:', error)
        this.showErrorMessage('加载收藏失败，请稍后重试')
      }
    },
    getMostUsedTag() {
      if (Object.keys(this.tagCounts).length === 0) {
        return '暂无'
      }
      const maxCount = Math.max(...Object.values(this.tagCounts))
      const mostUsedTags = Object.keys(this.tagCounts).filter(
        (tag) => this.tagCounts[tag] === maxCount,
      )
      return mostUsedTags[0] || '暂无'
    },
    showSuccessMessage(message) {
      const successDiv = document.createElement('div')
      successDiv.className = 'success-message'
      successDiv.textContent = message
      successDiv.style.cssText = `
        position: fixed;
        top: 20px;
        right: 20px;
        background: #4caf50;
        color: white;
        padding: 12px 20px;
        border-radius: 6px;
        box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        z-index: 10000;
        animation: slideInRight 0.3s ease;
      `
      document.body.appendChild(successDiv)
      setTimeout(() => {
        successDiv.style.animation = 'slideOutRight 0.3s ease'
        setTimeout(() => {
          if (successDiv.parentNode) {
            successDiv.parentNode.removeChild(successDiv)
          }
        }, 300)
      }, 3000)
    },
    showErrorMessage(message) {
      const errorDiv = document.createElement('div')
      errorDiv.className = 'error-message'
      errorDiv.textContent = message
      errorDiv.style.cssText = `
        position: fixed;
        top: 20px;
        right: 20px;
        background: #f44336;
        color: white;
        padding: 12px 20px;
        border-radius: 6px;
        box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        z-index: 10000;
        animation: slideInRight 0.3s ease;
      `
      document.body.appendChild(errorDiv)
      setTimeout(() => {
        errorDiv.style.animation = 'slideOutRight 0.3s ease'
        setTimeout(() => {
          if (errorDiv.parentNode) {
            errorDiv.parentNode.removeChild(errorDiv)
          }
        }, 300)
      }, 3000)
    },
    async changePage(page) {
      if (page < 1 || page > this.totalPages) return
      this.currentPage = page
    },
    async handleBookmarkClick(bookmark, event) {
      console.log('点击收藏链接:', bookmark.url)
      const bookmarkIndex = this.bookmarks.findIndex((b) => b.id === bookmark.id)
      if (bookmarkIndex !== -1) {
        this.bookmarks[bookmarkIndex].clickCount =
          (this.bookmarks[bookmarkIndex].clickCount || 0) + 1
      }
      this.sendClickRecordToBackend(bookmark.url)
      window.open(bookmark.url, '_blank', 'noopener,noreferrer')
      console.log('链接已打开:', bookmark.url)
    },
    async sendClickRecordToBackend(url) {
      try {
        recordBookmarkClick(url)
        console.log('点击记录已发送到服务器')
      } catch (error) {
        console.error('发送点击记录失败:', error)
      }
    },
    async handleDeleteBookmark(bookmark) {
      try {
        if (!confirm(`确定要删除收藏"${bookmark.title}"吗？`)) {
          return
        }
        const response = await deleteBookmark({
          url: bookmark.url,
          tag: bookmark.tags && bookmark.tags.length > 0 ? bookmark.tags[0] : 'default',
        })
        if (response.success) {
          this.showSuccessMessage('收藏删除成功！')
          const index = this.bookmarks.findIndex((b) => b.id === bookmark.id)
          if (index !== -1) {
            this.bookmarks.splice(index, 1)
          }
          this.totalBookmarksCount--
          await this.loadUserTags()
          if (this.bookmarks.length === 0 && this.currentPage > 1) {
            await this.changePage(this.currentPage - 1)
          }
          this.totalPages = Math.ceil(this.totalBookmarksCount / this.pageSize)
        } else {
          const errorMessage = response.message || '删除失败'
          alert(`删除失败：${errorMessage}`)
          this.showErrorMessage(errorMessage)
        }
      } catch (error) {
        console.error('删除收藏失败:', error)
        let errorMessage = '删除收藏失败，请稍后重试'
        if (error.message) {
          errorMessage = error.message
        }
        alert(`删除失败：${errorMessage}`)
        this.showErrorMessage(errorMessage)
      }
    },
    async handleLogout() {
      try {
        const token = localStorage.getItem('userToken') || 'fromLogin'
        const userId = localStorage.getItem('userId')
        if (userId) {
          try {
            await prefixTreeLogout(userId)
            console.log('用户缓存数据清除成功')
          } catch (error) {
            console.error('清除用户缓存数据失败:', error)
          }
        }
        const response = await logout(token)
        if (response.success) {
          this.showSuccessMessage('退出登录成功！')
          localStorage.removeItem('isLoggedIn')
          localStorage.removeItem('userEmail')
          localStorage.removeItem('userId')
          localStorage.removeItem('userData')
          localStorage.removeItem('user_info')
          localStorage.removeItem('auth_token')
          window.dispatchEvent(new CustomEvent('loginStatusChanged'))
          this.showUserCenter = false
        } else {
          this.showErrorMessage(response.message)
          if (response.message.includes('无效的token') || response.message.includes('用户不存在')) {
            localStorage.removeItem('isLoggedIn')
            localStorage.removeItem('userEmail')
            localStorage.removeItem('userId')
            localStorage.removeItem('userData')
            localStorage.removeItem('user_info')
            localStorage.removeItem('auth_token')
            window.dispatchEvent(new CustomEvent('loginStatusChanged'))
            this.showUserCenter = false
          }
        }
      } catch (error) {
        console.error('退出登录失败:', error)
        this.showErrorMessage('退出登录失败，请稍后重试')
      }
    },
    scrollToDisplayModule() {
      this.$nextTick(() => {
        const displayModule = document.querySelector('.display-module')
        if (displayModule) {
          displayModule.scrollIntoView({
            behavior: 'smooth',
            block: 'start',
          })
        }
      })
    },
    async sendAIMessage() {
      if (!this.aiInput.trim()) return
      const userMessage = {
        id: Date.now(),
        type: 'user',
        content: this.aiInput,
        timestamp: new Date().toISOString(),
      }
      this.chatMessages.push(userMessage)
      const messageToSend = this.aiInput
      this.aiInput = ''
      this.$nextTick(() => {
        this.$refs.aiAssistant.scrollToBottom()
      })
      try {
        const userId = localStorage.getItem('userId')
        const response = await chatWithAI(messageToSend, userId)
        if (response.success) {
          const aiMessage = {
            id: Date.now() + 1,
            type: 'ai',
            content: response.response,
            timestamp: response.timestamp,
          }
          this.chatMessages.push(aiMessage)
          this.$nextTick(() => {
            this.$refs.aiAssistant.scrollToBottom()
          })
        }
      } catch (error) {
        console.error('AI对话失败:', error)
        const errorMessage = {
          id: Date.now() + 1,
          type: 'ai',
          content: '抱歉，我暂时无法回答您的问题，请稍后重试。',
          timestamp: new Date().toISOString(),
        }
        this.chatMessages.push(errorMessage)
      }
    },
  },
}
</script>

<style scoped>
/* 全局重置 */
:global(body) {
  margin: 0;
  padding: 0;
  overflow-x: hidden;
  box-sizing: border-box;
}

:global(*),
:global(*::before),
:global(*::after) {
  box-sizing: inherit;
}

.main-view {
  width: 100%;
  min-height: 100vh;
  position: relative;
  overflow-x: hidden; /* 确保不会溢出 */
}

.main-interface {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  /* background-color: #E5E4E2; */
    background: transparent;
    /* 噪点层，透明度 5% */
    /* url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noiseFilter'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.65' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noiseFilter)' opacity='0.05'/%3E%3C/svg%3E"),
    linear-gradient(135deg, #e5e4e2 0%, #7d8ca3 100%);
  background-blend-mode: multiply, normal; */
  z-index: 1;
}

.sub-interface {
  position: relative;
  background: white;
  width: 100%;
  min-height: 100vh;
  top: 10vh;
  z-index: 2;
  padding: 40px 20px;
  /* border-radius: 20px 20px 0 0; */
  /* box-shadow: 0 -4px 20px rgba(0, 0, 0, 0.1); */
  transition: top 0.3s ease; /* 添加过渡效果 */
}

.toggle-interface-btn {
  position: absolute;
  top: 20px;
  right: 20px;
  padding: 10px 20px;
  background: #000;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 400;
  font-size: 1.2rem;
  font-family: 'ReadexPro', serif;
  z-index: 999;
  
  transition: background 0.3s ease;
}

.toggle-interface-btn:hover {
  background: #357abd;
}

@keyframes slideInRight {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideOutRight {
  from {
    transform: translateX(0);
    opacity: 1;
  }
  to {
    transform: translateX(100%);
    opacity: 0;
  }
}

@media (max-width: 768px) {
  .sub-interface {
    padding: 20px;
  }

  .toggle-interface-btn {
    top: 70px;
    right: 10px;
    padding: 8px 16px;
    font-size: 14px;
  }
}
</style>