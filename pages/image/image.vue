<template>
  <view class="container">
    <!-- 顶部翻译结果显示区域 -->
    <view class="result-container">
      <!-- 原始图片显示 -->
      <view class="image-section" v-if="imagePath">
        <image class="captured-image" :src="imagePath" mode="widthFix"></image>
      </view>
      
      <!-- 原始文字识别结果 -->
      <view class="original-section" v-if="originalText">
        <text class="section-title">识别结果</text>
        <view class="text-box">
          <text>{{ originalText }}</text>
        </view>
        <view class="language-info">
          <text>识别语言: {{ originalLanguage }}</text>
        </view>
      </view>
      
      <!-- 翻译结果 -->
      <view class="translated-section" v-if="translatedText">
        <text class="section-title">翻译结果</text>
        <view class="text-box">
          <text>{{ translatedText }}</text>
        </view>
        <view class="language-info">
          <text>目标语言: {{ targetLanguage }}</text>
        </view>
      </view>
      
      <!-- 无内容时的占位提示 -->
      <view class="placeholder" v-if="!imagePath && !originalText && !translatedText">
        <text>点击下方按钮拍摄照片，识别并翻译文字</text>
      </view>
    </view>
    
    <!-- 底部控制区域 -->
    <view class="control-container">
      <button class="capture-btn" @click="takePhoto">
        <uni-icons type="camera" size="30" color="#fff"></uni-icons>
        <text>拍摄照片</text>
      </button>
      
      <view class="language-selector">
        <picker mode="selector" :range="languageList" range-key="name" @change="onLanguageChange">
          <view class="picker">
            <text>翻译为: {{ targetLanguage }}</text>
            <uni-icons type="arrowdown" size="16"></uni-icons>
          </view>
        </picker>
      </view>
    </view>
    
    <!-- 加载指示器 -->
    <uni-load-more v-if="isProcessing" status="loading" :contentText="{
      contentdown: '处理中',
      contentrefresh: '处理中',
      contentnomore: '处理完成'
    }"></uni-load-more>
	<bottom :currentTab="'image'" />
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
      imagePath: '',
      originalText: '',
      translatedText: '',
      originalLanguage: '',
      targetLanguage: '英语',
      isProcessing: false,
      languageList: [
        { name: '英语', code: 'en' },
        { name: '中文', code: 'zh' },
        { name: '日语', code: 'ja' },
        { name: '韩语', code: 'ko' },
        { name: '法语', code: 'fr' },
        { name: '德语', code: 'de' },
        { name: '西班牙语', code: 'es' },
        { name: '俄语', code: 'ru' },
        { name: '阿拉伯语', code: 'ar' }
      ],
      selectedLangIndex: 0
    }
  },
  methods: {
    takePhoto() {
      uni.chooseImage({
        count: 1,
        sizeType: ['compressed'],
        sourceType: ['camera'],
        success: (res) => {
          this.imagePath = res.tempFilePaths[0];
          this.processImage(res.tempFilePaths[0]);
        },
        fail: (err) => {
          console.error('拍照失败', err);
          uni.showToast({
            title: '拍照失败',
            icon: 'none'
          });
        }
      });
    },
    
    processImage(imagePath) {
      this.isProcessing = true;
      this.originalText = '';
      this.translatedText = '';
      
      // 这里应该是调用OCR API的代码
      // 模拟OCR识别过程
      setTimeout(() => {
        this.originalText = '这是一个测试图像识别的结果';
        this.originalLanguage = '中文';
        this.isProcessing = false;
        
        // 自动翻译
        this.translateText();
      }, 2000);
    },
    
    translateText() {
      if (!this.originalText) return;
      
      this.isProcessing = true;
      
      // 这里应该是调用翻译API的代码
      // 模拟翻译结果
      setTimeout(() => {
        const translations = {
          'en': 'This is a test result of image recognition',
          'zh': '这是一个测试图像识别的结果',
          'ja': 'これは画像認識のテスト結果です',
          'ko': '이것은 이미지 인식의 테스트 결과입니다',
          'fr': 'Ceci est un résultat de test de reconnaissance d\'image',
          'de': 'Dies ist ein Testergebnis der Bilderkennung',
          'es': 'Este es un resultado de prueba de reconocimiento de imagen',
          'ru': 'Это результат теста распознавания изображений',
          'ar': 'هذه نتيجة اختبار التعرف على الصور'
        };
        
        const langCode = this.languageList[this.selectedLangIndex].code;
        this.translatedText = translations[langCode] || translations['en'];
        this.isProcessing = false;
      }, 1500);
    },
    
    onLanguageChange(e) {
      this.selectedLangIndex = e.detail.value;
      this.targetLanguage = this.languageList[this.selectedLangIndex].name;
      
      // 如果有原文，重新翻译
      if (this.originalText) {
        this.translateText();
      }
    }
  }
}
</script>

<style lang="scss">
.container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  padding: 20rpx;
  box-sizing: border-box;
  background-color: #f5f5f5;
}

.result-container {
  flex: 1;
  padding: 30rpx;
  background-color: #fff;
  border-radius: 20rpx;
  margin-bottom: 30rpx;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  
  .image-section {
    margin-bottom: 40rpx;
    
    .captured-image {
      width: 100%;
      border-radius: 15rpx;
      box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.1);
    }
  }
  
  .original-section, .translated-section {
    margin-bottom: 40rpx;
  }
  
  .section-title {
    font-size: 32rpx;
    color: #333;
    font-weight: bold;
    margin-bottom: 20rpx;
    display: block;
  }
  
  .text-box {
    background-color: #f9f9f9;
    padding: 30rpx;
    border-radius: 15rpx;
    margin-bottom: 15rpx;
    min-height: 120rpx;
    line-height: 1.6;
    white-space: pre-wrap;
  }
  
  .language-info {
    font-size: 26rpx;
    color: #666;
    text-align: right;
  }
  
  .placeholder {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #999;
    font-size: 30rpx;
    text-align: center;
    padding: 40rpx;
  }
}

.control-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20rpx 0;
  
  .capture-btn {
    width: 80%;
    height: 90rpx;
    background-color: #4a90e2;
    color: #fff;
    border-radius: 45rpx;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 30rpx;
    font-size: 32rpx;
    border: none;
    
    &:active {
      background-color: #3a7bc8;
    }
    
    text {
      margin-left: 15rpx;
    }
  }
  
  .language-selector {
    .picker {
      display: flex;
      align-items: center;
      color: #4a90e2;
      font-size: 30rpx;
      padding: 15rpx 30rpx;
      background-color: #f0f7ff;
      border-radius: 50rpx;
    }
  }
}

/* 响应式适配 */
@media (min-width: 768px) {
  .container {
    max-width: 600px;
    margin: 0 auto;
    padding: 30rpx;
  }
  
  .control-container {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    
    .capture-btn {
      width: 60%;
      margin-bottom: 0;
      margin-right: 30rpx;
    }
    
    .language-selector {
      flex: 1;
    }
  }
}
</style>