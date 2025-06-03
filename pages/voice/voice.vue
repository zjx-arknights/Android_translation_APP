<template>
  <!-- 模板部分保持不变 -->
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
            <!-- <uni-icons type="arrowdown" size="16"></uni-icons> -->
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
      recorderManager: null,
      tempFilePath: '' // 用于存储录音临时文件路径
    }
  },
  mounted() {
    // 初始化录音管理器
    this.recorderManager = uni.getRecorderManager();
    
    this.recorderManager.onStart(() => {
      console.log('录音开始');
    });
    
    this.recorderManager.onStop(async (res) => {
      console.log('录音结束', res);
      this.tempFilePath = res.tempFilePath;
      await this.handleRecordingResult(res);
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
      
      // 开始录音，设置符合百度API要求的参数
      this.recorderManager.start({
        format: 'wav', // 使用wav格式，百度API推荐
        sampleRate: 16000, // 16kHz采样率
        numberOfChannels: 1, // 单声道
        encodeBitRate: 256000, // 编码比特率
        frameSize: 4, // 帧大小
        duration: 60000 // 最长60秒
      });
    },
    
    stopRecording() {
      this.isRecording = false;
      this.recorderManager.stop();
    },
    
async handleRecordingResult(res) {
    try {
        uni.showLoading({ title: '翻译中...' });
        
        console.log('录音文件路径:', this.tempFilePath);

        if (!this.tempFilePath) {
            throw new Error('录音文件路径无效');
        }

        // 方式1：直接使用 UniApp 的文件读取（适用于临时文件）
        const fileData = await new Promise((resolve, reject) => {
            plus.io.resolveLocalFileSystemURL(
                this.tempFilePath,
                (entry) => {
                    entry.file((file) => {
                        const reader = new plus.io.FileReader();
                        reader.onload = (evt) => {
                            const base64Data = evt.target.result.split(',')[1]; // 提取纯base64
                            resolve({ data: base64Data });
                        };
                        reader.onerror = (err) => reject(err);
                        reader.readAsDataURL(file);
                    }, reject);
                },
                reject
            );
        });

        // 方式2：使用 Native.js（如果方式1失败）
        // const fileData = await this.readFileNative(this.tempFilePath);

        const targetLang = this.languageList[this.selectedLangIndex].code;
        
        const response = await uni.request({
            url: 'http://zhoujx.com:11454/api/translate/voice',
            method: 'POST',
            header: { 'Content-Type': 'application/json' },
            data: {
                from: 'auto',
                to: targetLang,
                voice: fileData.data,
                format: 'wav'
            }
        });

        const result = response[1].data;
        if (result.success) {
            this.originalText = result.originalText;
            this.originalLanguage = result.originalLanguage;
            this.translatedText = result.translatedText;
        } else {
            throw new Error(result.message || '翻译失败');
        }
    } catch (error) {
        console.error('处理录音出错:', error);
        uni.showToast({ title: error.message || '处理失败', icon: 'none' });
    } finally {
        uni.hideLoading();
    }
},

// 方式2：使用 Native.js 读取文件（如果 plus.io 不可用）
readFileNative(filePath) {
    return new Promise((resolve, reject) => {
        if (window.plus && plus.io) {
            plus.io.resolveLocalFileSystemURL(
                filePath,
                (entry) => {
                    entry.file((file) => {
                        const reader = new plus.io.FileReader();
                        reader.onload = (evt) => {
                            const base64Data = evt.target.result.split(',')[1];
                            resolve({ data: base64Data });
                        };
                        reader.onerror = reject;
                        reader.readAsDataURL(file);
                    }, reject);
                },
                reject
            );
        } else {
            reject(new Error('当前环境不支持文件读取'));
        }
    });
},

    onLanguageChange(e) {
      this.selectedLangIndex = e.detail.value;
      this.targetLanguage = this.languageList[this.selectedLangIndex].name;
      
      // 如果有原文，重新翻译
      if (this.originalText) {
        this.handleRecordingResult({ tempFilePath: this.tempFilePath });
      }
    },
    
    getRandomHeight() {
      return Math.floor(Math.random() * 20) + 10;
    }
  }
}
</script>

<!-- 样式部分保持不变 -->
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