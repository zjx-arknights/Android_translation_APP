<template>
  <view class="container">
    <!-- 顶部翻译结果显示区域 -->
    <view class="result-container">
      <view class="original-section" v-if="originalText">
        <text class="section-title">原始语音</text>
        <view class="text-box">
          <text>{{ originalText }}</text>
        </view>
        <view class="language-info">
          <text>识别语言: {{ originalLanguage }}</text>
        </view>
      </view>
      
      <view class="translated-section" v-if="translatedText">
        <text class="section-title">翻译结果</text>
        <view class="text-box">
          <text>{{ translatedText }}</text>
        </view>
        <view class="language-info">
          <text>目标语言: {{ targetLanguage }}</text>
        </view>
      </view>
      
      <view class="placeholder" v-if="!originalText && !translatedText">
        <text>按住下方按钮说话，松开后显示翻译结果</text>
      </view>
    </view>
    
    <!-- 底部录音按钮 -->
    <view class="control-container">
      <view 
        class="record-btn" 
        @touchstart="startRecording" 
        @touchend="stopRecording"
        :class="{ 'recording': isRecording }"
      >
        <view class="btn-inner">
          <text v-if="!isRecording">按住说话</text>
          <text v-else>松开结束</text>
        </view>
      </view>
      
      <view class="language-selector">
        <picker mode="selector" :range="languageList" range-key="name" @change="onLanguageChange">
          <view class="picker">
            <text>翻译为: {{ targetLanguage }}</text>
            <uni-icons type="arrowdown" size="16"></uni-icons>
          </view>
        </picker>
      </view>
    </view>
    
    <!-- 录音动画 -->
    <view class="recording-animation" v-if="isRecording">
      <view class="wave" v-for="(item, index) in 5" :key="index" :style="{ height: getRandomHeight() + 'px' }"></view>
    </view>
	<bottom :currentTab="'voice'" />
  </view>
</template>

<script>	
import bottom from '../need/bottom.vue';

export default {
	  components: {
	    bottom
	  },
	
  data() {
    return {
      isRecording: false,
      originalText: '',
      translatedText: '',
      originalLanguage: '',
      targetLanguage: '英语',
      languageList: [
        { name: '英语', code: 'en' },
        { name: '中文', code: 'zh' },
        { name: '日语', code: 'ja' },
        { name: '韩语', code: 'ko' },
        { name: '法语', code: 'fr' },
        { name: '德语', code: 'de' },
        { name: '西班牙语', code: 'es' },
      ],
      selectedLangIndex: 0,
      recorderManager: null
    }
  },
  mounted() {
    // 初始化录音管理器
    this.recorderManager = uni.getRecorderManager();
    
    this.recorderManager.onStart(() => {
      console.log('录音开始');
    });
    
    this.recorderManager.onStop((res) => {
      console.log('录音结束', res);
      this.handleRecordingResult(res);
    });
    
    this.recorderManager.onError((res) => {
      console.error('录音错误', res);
      uni.showToast({
        title: '录音失败',
        icon: 'none'
      });
      this.isRecording = false;
    });
  },
  methods: {
    startRecording() {
      this.isRecording = true;
      this.originalText = '';
      this.translatedText = '';
      
      // 开始录音
      this.recorderManager.start({
        format: 'mp3',
        duration: 60000 // 最长60秒
      });
    },
    
    stopRecording() {
      this.isRecording = false;
      this.recorderManager.stop();
    },
    
    handleRecordingResult(res) {
      // 这里应该是调用语音识别API的代码
      // 模拟识别结果
      setTimeout(() => {
        this.originalText = '这是一个测试语音识别的结果';
        this.originalLanguage = '中文';
        
        // 模拟翻译结果
        this.translateText();
      }, 1000);
    },
    
    translateText() {
      // 这里应该是调用翻译API的代码
      // 模拟翻译结果
      const translations = {
        'en': 'This is a test result of speech recognition',
        'zh': '这是一个测试语音识别的结果',
        'ja': 'これは音声認識のテスト結果です',
        'ko': '이것은 음성 인식의 테스트 결과입니다',
        'fr': 'Ceci est un résultat de test de reconnaissance vocale',
        'de': 'Dies ist ein Testergebnis der Spracherkennung',
        'es': 'Este es un resultado de prueba de reconocimiento de voz'
      };
      
      const langCode = this.languageList[this.selectedLangIndex].code;
      this.translatedText = translations[langCode] || translations['en'];
    },
    
    onLanguageChange(e) {
      this.selectedLangIndex = e.detail.value;
      this.targetLanguage = this.languageList[this.selectedLangIndex].name;
      
      // 如果有原文，重新翻译
      if (this.originalText) {
        this.translateText();
      }
    },
    
    getRandomHeight() {
      return Math.floor(Math.random() * 20) + 10;
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
  }
}

.control-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20rpx 0;
  
  .record-btn {
    width: 200rpx;
    height: 200rpx;
    border-radius: 50%;
    background-color: #4a90e2;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 40rpx;
    transition: all 0.2s;
    
    &.recording {
      background-color: #e74c3c;
      transform: scale(1.05);
    }
    
    .btn-inner {
      color: #fff;
      font-size: 32rpx;
      text-align: center;
    }
  }
  
  .language-selector {
    .picker {
      display: flex;
      align-items: center;
      color: #4a90e2;
      font-size: 30rpx;
    }
  }
}

.recording-animation {
  position: fixed;
  bottom: 300rpx;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  align-items: flex-end;
  height: 60rpx;
  
  .wave {
    width: 8rpx;
    background-color: #e74c3c;
    margin: 0 5rpx;
    border-radius: 4rpx;
    animation: wave 0.8s infinite alternate;
    
    @for $i from 1 through 5 {
      &:nth-child(#{$i}) {
        animation-delay: $i * 0.1s;
      }
    }
  }
}

@keyframes wave {
  from {
    height: 10rpx;
  }
  to {
    height: 50rpx;
  }
}

/* 响应式适配 */
@media (min-width: 768px) {
  .container {
    max-width: 500px;
    margin: 0 auto;
  }
}
</style>