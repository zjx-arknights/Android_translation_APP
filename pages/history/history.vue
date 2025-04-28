<template>
  <view class="container">
    <view class="header">
      <text class="title">翻译历史记录</text>
    </view>

    <view class="search-bar">
      <input 
        class="search-input" 
        v-model="searchText" 
        placeholder="搜索翻译内容..." 
        @confirm="searchHistory"
      />
      <button class="search-btn" @click="searchHistory">搜索</button>
    </view>

    <view class="filters">
      <view class="filter-group">
        <text class="filter-label">源语言:</text>
        <picker 
          class="filter-select" 
          mode="selector" 
          :range="languageOptions" 
          range-key="label"
          @change="onSourceLangChange"
        >
          <view class="picker-view">{{selectedSourceLang.label || '全部'}}</view>
        </picker>
      </view>

      <view class="filter-group">
        <text class="filter-label">目标语言:</text>
        <picker 
          class="filter-select" 
          mode="selector" 
          :range="languageOptions" 
          range-key="label"
          @change="onTargetLangChange"
        >
          <view class="picker-view">{{selectedTargetLang.label || '全部'}}</view>
        </picker>
      </view>

      <view class="filter-group">
        <text class="filter-label">日期:</text>
        <picker 
          class="filter-select" 
          mode="selector" 
          :range="dateOptions" 
          range-key="label"
          @change="onDateFilterChange"
        >
          <view class="picker-view">{{selectedDateFilter.label || '全部'}}</view>
        </picker>
      </view>
    </view>

    <view class="history-list">
      <view 
        v-for="(item, index) in filteredHistory" 
        :key="index" 
        class="history-item"
      >
        <view class="translation-pair">
          <view class="translation-box source">
            <text class="language-tag">{{item.sourceLang}}</text>
            <view class="text-content">{{item.sourceText}}</view>
          </view>
          <view class="translation-box target">
            <text class="language-tag">{{item.targetLang}}</text>
            <view class="text-content">{{item.targetText}}</view>
          </view>
        </view>
        <view class="meta-info">
          <text>{{item.date}}</text>
          <text>{{item.translationType}}</text>
        </view>
        <view class="actions">
          <button class="action-btn" @click="copyTranslation(index)">
            <uni-icons type="copy" size="16" color="#1890ff" style="margin-right: 5px;"></uni-icons>
            复制
          </button>
          <button class="action-btn" @click="deleteTranslation(index)">
            <uni-icons type="trash" size="16" color="#ff4d4f" style="margin-right: 5px;"></uni-icons>
            删除
          </button>
        </view>
      </view>

      <view v-if="filteredHistory.length === 0" class="no-results">
        <text>没有找到匹配的翻译记录</text>
      </view>
    </view>

    <view class="pagination">
      <button 
        class="page-btn" 
        :disabled="currentPage === 1" 
        @click="prevPage"
      >
        上一页
      </button>
      <text class="page-info">第 {{currentPage}} 页</text>
      <button 
        class="page-btn" 
        :disabled="currentPage >= totalPages" 
        @click="nextPage"
      >
        下一页
      </button>
    </view>
	<view>
		<!-- 底部导航栏 -->
		     <bottom :currentTab="'history'" />
	</view>
  </view>
</template>

<script>
	import bottom from '../need/bottom.vue';
export default {
	components: {
	  bottom,  // 在 components 中注册组件
	},
  data() {
    return {
      searchText: '',
      currentPage: 1,
      itemsPerPage: 10,
      selectedSourceLang: {},
      selectedTargetLang: {},
      selectedDateFilter: {},
      languageOptions: [
        { value: '', label: '全部' },
        { value: 'zh', label: '中文' },
        { value: 'en', label: '英语' },
        { value: 'ja', label: '日语' },
        { value: 'ko', label: '韩语' },
        { value: 'fr', label: '法语' },
        { value: 'de', label: '德语' },
        { value: 'es', label: '西班牙语' }
      ],
      dateOptions: [
        { value: '', label: '全部' },
        { value: 'today', label: '今天' },
        { value: 'week', label: '本周' },
        { value: 'month', label: '本月' }
      ],
      translationHistory: [
        {
          sourceLang: '中文 (zh)',
          sourceText: '你好，世界！这是一个翻译示例。',
          targetLang: '英语 (en)',
          targetText: 'Hello, world! This is a translation example.',
          date: '2023-06-15 14:30:22',
          translationType: '手动翻译'
        },
        {
          sourceLang: '英语 (en)',
          sourceText: 'The quick brown fox jumps over the lazy dog.',
          targetLang: '中文 (zh)',
          targetText: '敏捷的棕色狐狸跳过了懒惰的狗。',
          date: '2023-06-14 09:15:45',
          translationType: '语音翻译'
        },
        {
          sourceLang: '法语 (fr)',
          sourceText: 'Bonjour, comment allez-vous aujourd\'hui ?',
          targetLang: '英语 (en)',
          targetText: 'Hello, how are you today?',
          date: '2023-06-13 16:22:10',
          translationType: '图像翻译'
        }
      ]
    }
  },
  computed: {
    filteredHistory() {
      let filtered = this.translationHistory.filter(item => {
        // 搜索文本匹配
        const matchesSearch = this.searchText === '' || 
          item.sourceText.toLowerCase().includes(this.searchText.toLowerCase()) || 
          item.targetText.toLowerCase().includes(this.searchText.toLowerCase());
        
        // 源语言匹配
        const matchesSourceLang = !this.selectedSourceLang.value || 
          item.sourceLang.includes(`(${this.selectedSourceLang.value})`);
        
        // 目标语言匹配
        const matchesTargetLang = !this.selectedTargetLang.value || 
          item.targetLang.includes(`(${this.selectedTargetLang.value})`);
        
        // 日期匹配 (简化逻辑，实际应用中应比较实际日期)
        const matchesDate = !this.selectedDateFilter.value || 
          (this.selectedDateFilter.value === 'today' && item.date.includes('2023-06-15')) ||
          (this.selectedDateFilter.value === 'week' && (item.date.includes('2023-06-1') || item.date.includes('2023-06-0'))) ||
          (this.selectedDateFilter.value === 'month' && item.date.includes('2023-06'));
        
        return matchesSearch && matchesSourceLang && matchesTargetLang && matchesDate;
      });
      
      // 分页处理
      const start = (this.currentPage - 1) * this.itemsPerPage;
      return filtered.slice(start, start + this.itemsPerPage);
    },
    totalPages() {
      return Math.ceil(this.translationHistory.length / this.itemsPerPage);
    }
  },
  methods: {
    searchHistory() {
      this.currentPage = 1; // 搜索时重置到第一页
    },
    onSourceLangChange(e) {
      this.selectedSourceLang = this.languageOptions[e.detail.value];
      this.searchHistory();
    },
    onTargetLangChange(e) {
      this.selectedTargetLang = this.languageOptions[e.detail.value];
      this.searchHistory();
    },
    onDateFilterChange(e) {
      this.selectedDateFilter = this.dateOptions[e.detail.value];
      this.searchHistory();
    },
    copyTranslation(index) {
      const item = this.filteredHistory[index];
      uni.setClipboardData({
        data: item.targetText,
        success: () => {
          uni.showToast({
            title: '翻译已复制到剪贴板',
            icon: 'none'
          });
        }
      });
    },
    reuseTranslation(index) {
      const item = this.filteredHistory[index];
      uni.showModal({
        title: '复用翻译',
        content: `源语言: ${item.sourceLang}\n目标语言: ${item.targetLang}\n原文: ${item.sourceText}\n译文: ${item.targetText}`,
        showCancel: false
      });
    },
    deleteTranslation(index) {
      uni.showModal({
        title: '提示',
        content: '确定要删除这条翻译记录吗？',
        success: (res) => {
          if (res.confirm) {
            // 在实际应用中，这里应该调用API删除服务器上的记录
            // 这里只是从前端列表中移除
            const item = this.filteredHistory[index];
            this.translationHistory = this.translationHistory.filter(
              i => !(i.sourceText === item.sourceText && i.targetText === item.targetText)
            );
            uni.showToast({
              title: '删除成功',
              icon: 'success'
            });
          }
        }
      });
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    }
  }
}
</script>

<style>
.container {
  padding: 20px;
  background-color: #f5f5f5;
}

.header {
  margin-bottom: 30px;
}

.title {
  text-align: center;
  font-size: 20px;
  color: #2c3e50;
  font-weight: bold;
}

.search-bar {
  display: flex;
  margin-bottom: 20px;
}

.search-input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  background-color: #fff;
}

.search-btn {
  margin-left: 10px;
  padding: 0 20px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  line-height: 2.3;
}

.filters {
  display: flex;
  flex-wrap: wrap;
  margin-bottom: 20px;
}

.filter-group {
  display: flex;
  align-items: center;
  margin-right: 15px;
  margin-bottom: 10px;
}

.filter-label {
  font-weight: bold;
  margin-right: 5px;
}

.filter-select {
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 8px;
  background-color: #fff;
}

.picker-view {
  height: 100%;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.history-item {
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 15px;
  background-color: #fff;
}

.translation-pair {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 10px;
}

.translation-box {
  padding: 10px;
  border-radius: 4px;
}

.source {
  background-color: #f8f9fa;
  border-left: 4px solid #3498db;
}

.target {
  background-color: #e8f4fd;
  border-left: 4px solid #2ecc71;
}

.language-tag {
  display: inline-block;
  padding: 2px 8px;
  background-color: #3498db;
  color: white;
  border-radius: 12px;
  font-size: 12px;
  margin-bottom: 8px;
}

.target .language-tag {
  background-color: #2ecc71;
}

.text-content {
  white-space: pre-wrap;
  word-break: break-word;
}

.meta-info {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  color: #7f8c8d;
  margin-top: 10px;
}

.actions {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.action-btn {
  padding: 5px 10px;
  background-color: #ecf0f1;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 30px;
}

.page-btn {
  padding: 8px 15px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 14px;
}

.page-btn:disabled {
  background-color: #bdc3c7;
}

.page-info {
  margin: 0 15px;
}

.no-results {
  text-align: center;
  padding: 40px;
  color: #7f8c8d;
}

@media (min-width: 768px) {
  .translation-pair {
    flex-direction: row;
  }
  
  .translation-box {
    flex: 1;
  }
}

.actions {
  display: flex;
  gap: 12px;
  margin-top: 15px;
  justify-content: flex-end;
}

.action-btn {
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  background-color: #fff;
  border: 1px solid #eee;
}

.action-btn:active {
  transform: translateY(1px);
}

/* 复制按钮 */
.action-btn:nth-child(1) {
  color: #1890ff;
  border-color: #d0e3ff;
}

.action-btn:nth-child(1):active {
  background-color: #f0f7ff;
}

/* 删除按钮 */
.action-btn:nth-child(2) {
  color: #ff4d4f;
  border-color: #ffccc7;
}

.action-btn:nth-child(2):active {
  background-color: #fff0f0;
}
</style>