<template>
  <view class="container">
    <!-- 顶部标题 -->
    <view class="header">
      <text class="app-title">BabelBreak</text>
      <text class="app-subtitle">智能多语言翻译</text>
    </view>

    <!-- 语言选择器 -->
    <view class="language-selector">
      <picker class="language-select" mode="selector" :range="languageOptions" range-key="label" @change="changeSourceLanguage">
        <view class="language-select-content left_">
          <image class="language-icon" :src="getLanguageIcon(sourceLanguage)" mode="aspectFit"></image>
          <text>{{ sourceLanguageLabel }}</text>
          <uni-icons type="zh" size="20" color="#666"></uni-icons>
        </view>
      </picker>
      
      <view class="language-swap" @click="swapLanguages">
        <uni-icons type="reload" size="20" color="#1890ff"></uni-icons>
      </view>
      
      <picker class="language-select" mode="selector" :range="languageOptions" range-key="label" @change="changeTargetLanguage">
        <view class="language-select-content">
          <image class="language-icon" :src="getLanguageIcon(targetLanguage)" mode="aspectFit"></image>
          <text>{{ targetLanguageLabel }}</text>
          <uni-icons type="en" size="20" color="#666"></uni-icons>
        </view>
      </picker>
    </view>

    <!-- 输入区域 -->
    <view class="input-area">
      <textarea 
        v-model="inputText" 
        :placeholder="getPlaceholder()" 
        class="textarea"
        placeholder-class="textarea-placeholder"
        auto-height
      ></textarea>
      <view v-if="inputText" class="clear-btn" @click="clearInput">
        <uni-icons type="clear" size="20" color="#999"></uni-icons>
      </view>
    </view>

    <!-- 翻译按钮 -->
    <button @click="translateText" :disabled="isTranslating || !inputText.trim()" class="translate-btn" :loading="isTranslating">
      {{ isTranslating ? '翻译中...' : '翻译' }}
      <uni-icons v-if="!isTranslating" type="paperplane" size="16" color="#fff" style="margin-left: 5px;"></uni-icons>
    </button>

    <!-- 加载状态 -->
    <view v-if="isTranslating" class="loading">
      <uni-icons type="spinner-cycle" size="20" color="#1890ff" class="loading-icon"></uni-icons>
      <text>正在翻译，请稍候...</text>
    </view>

    <!-- 错误提示 -->
    <view v-if="error" class="error">
      <uni-icons type="info" size="16" color="#f5222d" style="margin-right: 5px;"></uni-icons>
      <text>{{ error }}</text>
    </view>

    <!-- 翻译结果 -->
    <view v-if="translatedText" class="result-container">
      <view class="result-header">
        <text class="result-title">翻译结果</text>
        <view class="result-actions">
          <view class="result-action" @click="copyResult">
            <uni-icons type="copy" size="16" color="#666"></uni-icons>
            <text>复制</text>
          </view>
          <view class="result-action" @click="speakResult">
            <uni-icons type="sound" size="16" color="#666"></uni-icons>
            <text>朗读</text>
          </view>
        </view>
      </view>
      <scroll-view scroll-y class="result-text-container">
        <text class="result-text">{{ translatedText }}</text>
      </scroll-view>
    </view>

    <!-- 底部导航栏 -->
    <bottom :currentTab="'home'" />
  </view>
</template>

<script>
import bottom from '../need/bottom.vue';
import uniIcons from '../need/icons.vue';

export default {
  components: {
    bottom,
    uniIcons
  },
  data() {
    return {
      inputText: '',
      translatedText: '',
      isTranslating: false,
      error: '',
      sourceLanguage: 'zh',
      targetLanguage: 'en',
      languageOptions: [
        { value: 'zh', label: '中文' },
        { value: 'en', label: '英文' },
        { value: 'jp', label: '日语' },
        { value: 'wyw', label: '文言文' },
        { value: 'kor', label: '韩语' },
        { value: 'fra', label: '法语' },
        { value: 'ara', label: '阿拉伯语' },
        { value: 'ru', label: '俄语' }
      ],
      languageIcons: {
        zh: '/static/languages/chinese.png',
        en: '/static/languages/english.png',
        jp: '/static/languages/japanese.png',
        wyw: '/static/languages/chinese.png',
        kor: '/static/languages/korean.png',
        fra: '/static/languages/french.png',
        ara: '/static/languages/arabic.png',
        ru: '/static/languages/russian.png'
      }
    };
  },
  computed: {
    sourceLanguageLabel() {
      return this.languageOptions.find(item => item.value === this.sourceLanguage)?.label || '中文';
    },
    targetLanguageLabel() {
      return this.languageOptions.find(item => item.value === this.targetLanguage)?.label || '英文';
    }
  },
  methods: {
    getLanguageIcon(lang) {
      return this.languageIcons[lang] || '/static/languages/default.png';
    },
    changeSourceLanguage(e) {
      this.sourceLanguage = this.languageOptions[e.detail.value].value;
    },
    changeTargetLanguage(e) {
      this.targetLanguage = this.languageOptions[e.detail.value].value;
    },
    swapLanguages() {
      [this.sourceLanguage, this.targetLanguage] = [this.targetLanguage, this.sourceLanguage];
    },
    getPlaceholder() {
      const languageMap = {
        zh: '请输入要翻译的中文内容...',
        en: 'Please enter the English text to translate...',
        jp: '翻訳したい日本語の内容を入力してください...',
        wyw: '请输入要翻译的文言文内容...',
        kor: '번역할 한국어 내용을 입력하세요...',
        fra: 'Veuillez entrer le texte français à traduire...',
        ara: 'الرجاء إدخال النص العربي للترجمة...',
        ru: 'Пожалуйста, введите текст на русском для перевода...'
      };
      return languageMap[this.sourceLanguage] || '请输入要翻译的内容...';
    },
    clearInput() {
      this.inputText = '';
      this.clearResult();
    },
    async translateText() {
      if (!this.inputText.trim()) {
        const errorMessages = {
          zh: '请输入要翻译的内容',
          en: 'Please enter text to translate',
          jp: '翻訳するテキストを入力してください',
          wyw: '请输入要翻译的内容',
          kor: '번역할 텍スト를 입력하세요',
          fra: 'Veuillez entrer le texte à traduire',
          ara: 'الرجاء إدخال النص للترجمة',
          ru: 'Пожалуйста, введите текст для перевода'
        };
        this.showError(errorMessages[this.sourceLanguage] || '请输入要翻译的内容');
        return;
      }
    
      this.clearError();
      this.clearResult();
      this.isTranslating = true;
    
      try {
        const response = await new Promise((resolve, reject) => {
          uni.request({
            url: 'http://zhoujx.com:11454/api/translate',
            method: 'POST',
            data: {
              text: this.inputText,
              from: this.sourceLanguage,
              to: this.targetLanguage,
            },
            header: {
              'Content-Type': 'application/json',
            },
            success: (res) => resolve(res),
            fail: (err) => reject(err)
          });
        });
        
        if (response.statusCode !== 200) {
          throw new Error(response.data?.error || '请求失败');
        }
    
        if (response.data.result) {
          this.translatedText = response.data.result;
        } else {
          const errorMessages = {
            zh: '未获取到翻译结果',
            en: 'Failed to get translation',
            jp: '翻訳結果を取得できませんでした',
            wyw: '未获取到翻译结果',
            kor: '번역結果를 가져오지 못했습니다',
            fra: 'Échec de la traduction',
            ara: 'فشل الحصول على الترجمة',
            ru: 'Не удалось получить перевод'
          };
          this.showError(errorMessages[this.sourceLanguage] || '未获取到翻译结果');
        }
      } catch (err) {
        const errorMessages = {
          zh: '翻译失败，请稍后再试',
          en: 'Translation failed, please try again later',
          jp: '翻訳に失敗しました。後でもう一度お試しください',
          wyw: '翻译失败，请稍后再试',
          kor: '번역에 실패했습니다. 나중에 다시 시도해 주세요',
          fra: 'La traduction a échoué, veuillez réessayer plus tard',
          ara: 'فشل الترجمة، يرجى المحاولة مرة أخرى لاحقًا',
          ru: 'Ошибка перевода, пожалуйста, попробуйте позже'
        };
        this.showError(errorMessages[this.sourceLanguage] || '翻译失败，请稍后再试');
        console.error('翻译错误:', err);
      } finally {
        this.isTranslating = false;
      }
    },
    copyResult() {
      uni.setClipboardData({
        data: this.translatedText,
        success: () => {
          uni.showToast({
            title: '复制成功',
            icon: 'success'
          });
        }
      });
    },
    speakResult() {
      uni.showToast({
        title: '朗读功能开发中',
        icon: 'none'
      });
    },
    showError(message) {
      this.error = message;
      uni.showToast({
        title: message,
        icon: 'none',
        duration: 2000
      });
    },
    clearError() {
      this.error = '';
    },
    clearResult() {
      this.translatedText = '';
    },
  },
};
</script>

<style lang="scss" scoped>
.container {
  padding: 20px;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #e4e8eb 100%);
  display: flex;
  flex-direction: column;
}

.header {
  margin-bottom: 24px;
  text-align: center;
}

.app-title {
  font-size: 24px;
  font-weight: bold;
  color: #1890ff;
  display: block;
}

.app-subtitle {
  font-size: 14px;
  color: #666;
  margin-top: 5px;
  display: block;
}

.language-selector {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
  background-color: #fff;
  border-radius: 12px;
  padding: 12px 16px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.language-select {
  flex: 1;
}

.language-select-content {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
}

.language-icon {
  width: 24px;
  height: 24px;
  margin-right: 8px;
}

.language-swap {
  padding: 0 12px;
  // margin-left: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.input-area {
  position: relative;
  margin-bottom: 20px;
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.textarea {
  width: 100%;
  min-height: 150px;
  padding: 16px;
  font-size: 16px;
  border-radius: 12px;
  box-sizing: border-box;
  border: none;
}

.textarea-placeholder {
  color: #999;
}

.clear-btn {
  position: absolute;
  right: 12px;
  bottom: 12px;
  padding: 5px;
  background-color: #f5f5f5;
  border-radius: 50%;
}

.translate-btn {
  background: linear-gradient(135deg, #1890ff 0%, #096dd9 100%);
  color: white;
  border: none;
  padding: 0 24px;
  height: 48px;
  line-height: 48px;
  font-size: 16px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(24, 144, 255, 0.3);
  // display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  transition: all 0.3s;
}

.translate-btn[disabled] {
  background: #d9d9d9;
  color: #666;
  box-shadow: none;
}

.loading {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #1890ff;
  margin: 20px 0;
  font-size: 14px;
}

.loading-icon {
  animation: rotate 1s linear infinite;
  margin-right: 8px;
}

.error {
  display: flex;
  align-items: center;
  color: #f5222d;
  margin: 20px 0;
  padding: 12px;
  background-color: #fff1f0;
  border-radius: 8px;
  font-size: 14px;
}

.result-container {
  flex: 1;
  background-color: #fff;
  border-radius: 12px;
  padding: 16px;
  margin-bottom: 100px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.result-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.result-title {
  font-size: 18px;
  font-weight: bold;
  color: #1890ff;
}

.result-actions {
  display: flex;
}

.result-action {
  display: flex;
  align-items: center;
  margin-left: 12px;
  color: #666;
  font-size: 14px;
  cursor: pointer;
}

.result-action text {
  margin-left: 4px;
}

.result-text-container {
  max-height: 200px;
}

.result-text {
  font-size: 16px;
  line-height: 1.6;
  color: #333;
}

.left_{
	padding-right: 40px;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
</style>