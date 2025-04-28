<template>
  <view class="tab-bar">
    <view class="tab-item" @click="switchTab('home')">
      <image class="tab-icon" :src="currentTab === 'home' ? '/static/tabs/home-active.png' : '/static/tabs/home.png'"></image>
      <text :class="['tab-text', currentTab === 'home' ? 'active' : '']" @click="switchTab('home')">首页</text>
    </view>
    <view class="tab-item" @click="switchTab('voice')">
      <image class="tab-icon" :src="currentTab === 'voice' ? '/static/tabs/translate-active.png' : '/static/tabs/translate.png'"></image>
      <text :class="['tab-text', currentTab === 'voice' ? 'active' : '']" @click="switchTab('voice')">语音翻译</text>
    </view>
    <view class="tab-item" @click="switchTab('image')">
      <image class="tab-icon" :src="currentTab === 'image' ? '/static/tabs/image-active.png' : '/static/tabs/image.png'"></image>
      <text :class="['tab-text', currentTab === 'image' ? 'active' : '']" @click="switchTab('image')">图像识别</text>
    </view>
    <view class="tab-item" @click="switchTab('history')">
      <image class="tab-icon" :src="currentTab === 'history' ? '/static/tabs/history-active.png' : '/static/tabs/history.png'"></image>
      <text :class="['tab-text', currentTab === 'history' ? 'active' : '']" @click="switchTab('history')">历史记录</text>
    </view>
  </view>
</template>

<script>
export default {
  props: {
    currentTab: {
      type: String,
      default: 'home'
    }
  },
  methods: {
    switchTab(tabName) {
      if (this.currentTab !== tabName) {
        uni.redirectTo({
          url: `/pages/${tabName}/${tabName}`
        });
      }
    }
  }
};
</script>

<style scoped>
.tab-bar {
  position: fixed;
  display: flex;
  bottom: 20px;
  left: 5vw;
  height: 50px;
  width: 90vw;
  line-height: 50px;
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.tab-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.tab-icon {
  width: 24px;
  height: 24px;
  margin-bottom: 2px;
}

.tab-text {
  font-size: 12px;
  color: #666;
}

.tab-text.active {
  color: #1890ff;
  font-weight: bold;
}
</style>
